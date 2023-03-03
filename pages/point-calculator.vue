<script setup lang="ts">
useHead({
    htmlAttrs: { "data-theme": "mytheme" },
});

// 門前か
const isConcealedHand = ref(false);
// 自摸か
const isPick = ref(false);
// 平和か
const isAllRuns = ref(false);
// 七対子か
const isSevenPairs = ref(false);

//門前加符・自摸符
const concealedPoints = computed((): number => {
    if (isPick.value) {
        // 自摸符
        return 2;
    } else {
        if (isConcealedHand.value) {
            // 門前加符
            return 10;
        } else {
            return 0;
        }
    }
});

const memberPointsList = ref([0, 0, 0, 0]);
const memberPoints = computed(() => {
    return memberPointsList.value.reduce((a, b) => {
        return a + b;
    });
});

// 雀頭
// 役牌
const isValueTiles = ref(false);
// 連風牌
const isOwnAndRoundWind = ref(false);
const headPoints = computed((): number => {
    // 雀頭
    if (isValueTiles.value) {
        if (isOwnAndRoundWind.value) {
            return 4;
        } else {
            return 2;
        }
    } else {
        return 0;
    }
});

// 待ち
const currentWaitType = ref<WaitType>("リャンメン");

const waitTypeList: Array<WaitType> = [
    "リャンメン",
    "カンチャン",
    "ペンチャン",
    "シャンポン",
    "タンキ",
    "ノベタン",
];
type WaitType = "リャンメン" | "カンチャン" | "ペンチャン" | "シャンポン" | "タンキ" | "ノベタン";
const waitPoints = computed((): number => {
    if (["ペンチャン", "カンチャン", "タンキ", "ノベタン"].includes(currentWaitType.value)) {
        return 2;
    } else {
        return 0;
    }
});

const sumPoints = computed(() => {
    // 副底
    let point = 20;

    //門前加符・自摸符
    point += concealedPoints.value;

    // 面子
    point += memberPoints.value;

    // 雀頭
    point += headPoints.value;

    // 待ち
    point += waitPoints.value;
    return point;
});

const totalPoints = computed(() => {
    // 七対子
    if (isSevenPairs.value) return 25;

    // 平和自摸
    if (isAllRuns.value && isPick.value) return 20;

    let point = sumPoints.value;

    point = Math.ceil(point / 10) * 10;
    return point;
});
</script>

<template>
    <div class="p-2">
        <p class="text-2xl">麻雀 符計算機</p>
        <div class="flex flex-col">
            <PointTypeHeader>※例外</PointTypeHeader>
            <div class="flex flex-row">
                <div class="grow">
                    <div class="w-full flex flex-row">
                        <ToggleButton :isActive="isAllRuns" @click="isAllRuns = !isAllRuns">
                            ピンフツモ
                        </ToggleButton>
                        <ToggleButton
                            :isActive="isSevenPairs"
                            @click="isSevenPairs = !isSevenPairs"
                        >
                            チートイツ
                        </ToggleButton>
                    </div>
                </div>
            </div>

            <template v-if="!isAllRuns && !isSevenPairs">
                <div class="divider"></div>

                <PointTypeHeader>①副底</PointTypeHeader>
                <div class="flex flex-row">
                    <div class="grow">
                        <div class="btn btn-block btn-disabled">20</div>
                    </div>
                    <PointTypeFooter>20</PointTypeFooter>
                </div>

                <div class="divider">+</div>

                <PointTypeHeader>②門前ツモ</PointTypeHeader>
                <div class="flex flex-row">
                    <div class="btn-group grow">
                        <ToggleButton :isActive="isPick" @click="isPick = true">
                            ツモ
                        </ToggleButton>
                        <ToggleButton :isActive="!isPick" @click="isPick = false">
                            ロン
                        </ToggleButton>
                    </div>
                    <ToggleButton
                        :isActive="isConcealedHand"
                        @click="isConcealedHand = !isConcealedHand"
                    >
                        メンゼン
                    </ToggleButton>
                    <PointTypeFooter> {{ concealedPoints }}</PointTypeFooter>
                </div>

                <div class="divider">+</div>

                <PointTypeHeader>③面子</PointTypeHeader>
                <div class="flex flex-row">
                    <div class="grow flex flex-col">
                        <template v-for="index in 4">
                            <ElementsPointCalculator
                                v-model="memberPointsList[index - 1]"
                            ></ElementsPointCalculator>
                        </template>
                    </div>
                    <PointTypeFooter>{{ memberPoints }}</PointTypeFooter>
                </div>

                <div class="divider">+</div>

                <PointTypeHeader>④雀頭</PointTypeHeader>
                <div class="flex flex-row">
                    <div class="grow flex flex-row">
                        <ToggleButton :isActive="isValueTiles" @click="isValueTiles = !isValueTiles"
                            >役牌</ToggleButton
                        >
                        <ToggleButton
                            :isActive="isOwnAndRoundWind"
                            :isDisabled="!isValueTiles"
                            @click="isOwnAndRoundWind = !isOwnAndRoundWind"
                            >連風牌</ToggleButton
                        >
                    </div>
                    <PointTypeFooter>{{ headPoints }}</PointTypeFooter>
                </div>

                <div class="divider">+</div>

                <PointTypeHeader>⑤待ち</PointTypeHeader>
                <div class="flex flex-row">
                    <div class="grow">
                        <div class="btn-group w-full">
                            <ToggleButton
                                v-for="waitType in waitTypeList"
                                :isActive="waitType === currentWaitType"
                                @click="currentWaitType = waitType"
                            >
                                {{ waitType }}
                            </ToggleButton>
                        </div>
                    </div>
                    <PointTypeFooter>{{ waitPoints }}</PointTypeFooter>
                </div>
            </template>

            <div class="divider">=</div>

            <PointTypeHeader>合計符</PointTypeHeader>
            <div class="flex flex-row">
                <div class="w-full text-center border-2">
                    <p class="text-2xl">{{ totalPoints }}符</p>
                    <p>
                        ①{{ 20 }}+②{{ concealedPoints }}+③{{ memberPoints }}+④{{ headPoints }}+⑤{{
                            waitPoints
                        }}={{ sumPoints }}
                    </p>
                </div>
            </div>
        </div>
        <button class="btn">リセット</button>
    </div>
</template>
