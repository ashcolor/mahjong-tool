<script setup lang="ts">
interface Props {
    modelValue: number;
}

const props = withDefaults(defineProps<Props>(), {
    modelValue: 0,
});

interface Emits {
    (e: "update:modelValue", points: number): void;
}

const emit = defineEmits<Emits>();

const memberTypeList = ["順子", "刻子", "槓子"];

const currentMemberType = ref("順子");
const isSimple = ref(true);
const isConcealed = ref(true);

const points = computed<number>(() => {
    if (currentMemberType.value === "順子") {
        return 0;
    }

    let sumPoints = 0;

    if (currentMemberType.value === "刻子") {
        sumPoints = 2;
    }
    if (currentMemberType.value === "槓子") {
        sumPoints = 4;
    }

    if (!isSimple.value) {
        sumPoints *= 2;
    }

    if (isConcealed.value) {
        sumPoints *= 2;
    }

    emit("update:modelValue", sumPoints);
    return sumPoints;
});
</script>

<template>
    <div class="flex flex-row">
        <div class="btn-group">
            <ToggleButton
                v-for="memberType in memberTypeList"
                :isActive="memberType === currentMemberType"
                @click="currentMemberType = memberType"
            >
                {{ memberType }}
            </ToggleButton>
        </div>
        <div class="btn-group">
            <ToggleButton
                :isActive="isSimple"
                :isDisabled="currentMemberType === '順子'"
                @click="isSimple = true"
            >
                中張牌
            </ToggleButton>
            <ToggleButton
                :isActive="!isSimple"
                :isDisabled="currentMemberType === '順子'"
                @click="isSimple = false"
            >
                幺九牌
            </ToggleButton>
        </div>
        <div class="btn-group">
            <ToggleButton :isActive="isConcealed" @click="isConcealed = true"> 暗刻 </ToggleButton>
            <ToggleButton :isActive="!isConcealed" @click="isConcealed = false">
                明刻
            </ToggleButton>
        </div>
        <div class="grow grid place-items-center">{{ points }}符</div>
    </div>
</template>
