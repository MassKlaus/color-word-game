<template>
    <div class="h-screen p-4 grid grid-rows-[50px_1fr] items-center bg-gray-50 dark:bg-slate-950">
        <div class="flex justify-between items-center">
            <ModeToggle />
            <div v-if="playing" :class="randomValue(values.color)" class="text-xl" :key="goal">
                {{ properText(goalText ?? "") }}
            </div>
            <Button v-if="playing" @click="stopGame" variant="destructive">
                Quit
            </Button>
        </div>
        <div class="flex justify-center items-center" v-if="!playing">
            <Button @click="startGame">Start The Game</Button>
        </div>
        <div ref="gameArea" v-else class="h-full w-full">
            <div class="relative">
                <Button v-for="x in targets" :class="cn(x.color, x.size, `hover:${x.color}`, 'absolute')" :style="{
                    top: x.y + 'px',
                    left: x.x + 'px',
                }" variant="ghost" @click="() => clickTarget(x)">
                    {{ properText(x.text) }}
                </Button>
            </div>
        </div>
    </div>
</template>

<script setup lang="tsx">
import type { VNode } from 'vue';
import { Button } from '~/components/ui/button';
import { cn } from '~/lib/utils';


const playing = ref(false)

const values = {
    size: [
        "text-sm",
        "text-md",
        "text-lg",
        "text-xl",
        "text-xxl",
        "text-xxxl",
    ],
    color: [
        "text-red-500",
        "text-green-500",
        "text-blue-500",
        "text-purple-500",
        "text-foreground",
    ],
    text: [
        "Red",
        "Green",
        "Blue",
        "Purple",
        "Foreground",
    ],
} as const;

type TextTargetValues = typeof values;

type TextTarget = {
    color: TextTargetValues["color"][number],
    size: TextTargetValues["size"][number],
    text: TextTargetValues["text"][number],
    x: number,
    y: number,
}

const targets = ref<TextTarget[]>([]);
const goal = ref<TextTargetValues["color"][number] | undefined>("text-red-500")
const goalText = computed(() => {
    if (goal.value == null) {
        return "";
    }

    const colorIndex = values.color.indexOf(goal.value);

    if (colorIndex < 0) {
        return "";
    }

    return properText(values.text[colorIndex]);
})

const round = ref(0);
const mistake = ref(0);

const colorMode = useColorMode();

function properText(text: string) {
    if (text == "Foreground") {
        return colorMode.value == "dark" ? "White" : "Black";
    }

    return text;
}

function startGame() {
    playing.value = true;
    startRound();
}

function startRound() {
    if (gameArea.value == null) {
        nextTick(startRound);
        return;
    }

    generateTargets();
}

function stopGame() {
    playing.value = false;
    targets.value = []
    goal.value = undefined;
}

const totaltargets = 10;
const gameArea = ref<HTMLElement>();

function generateTargets() {
    if (gameArea.value == null) {
        return;
    }

    const width = gameArea.value.clientWidth;
    const height = gameArea.value.clientHeight;

    const targetGoal = randomValue(values.color)
    goal.value = targetGoal;

    const newTargets: TextTarget[] = [];
    for (let index = 0; index < totaltargets; index++) {
        let x = width * Math.random()
        let y = height * Math.random()

        if (x < 100) {
            x += 100;
        }

        if (x > width - 100) {
            x -= 100;
        }

        if (y < 100) {
            y += 100;
        }

        if (y > height - 100) {
            y -= 100;
        }

        let color = randomValue(values.color);

        while (color == targetGoal)
            color = randomValue(values.color);

        const element: TextTarget = {
            color: randomValue(values.color),
            size: randomValue(values.size),
            text: randomValue(values.text),
            x: x,
            y: y,
        }

        newTargets.push(element);
    }

    const randomTarget = randomValue(newTargets);
    console.log(randomTarget);
    randomTarget.color = targetGoal;

    targets.value = newTargets;
    avoidCollisions();
}

function avoidCollisions() {
    let collision = false;

    for (const target of targets.value) {
        for (const button of targets.value) {
            if (button == target) {
                continue;
            }

            const length = Math.abs(Math.sqrt(button.x * button.x + button.y * button.y) - Math.sqrt(target.x * target.x + target.y * target.y))

            if (length < 30) {
                button.x += 20;
                button.y += 20;
                target.x -= 20;
                target.y -= 20;
                collision = true;
            }            
        }
    }

    if (collision) {
        avoidCollisions();
    }
}

function randomValue(list: any) {
    return list[Math.floor(Math.random() * list.length)];
}

function clickTarget(target: TextTarget) {
    if (target.color == goal.value) {
        // start next round
        round.value += 1;
        startRound();
    }

    // up mistake count
    mistake.value += 1;
    console.log("Mistake");
}



</script>