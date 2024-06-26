<script setup lang="ts">
import { computed, ref } from "vue";
import { useGlobalStore } from "../../store/global";
import { KeySteeringWheel } from "../../keyMappingConfig";
import { NButton, NFormItem, NH4, NIcon, NInput, NInputNumber } from "naive-ui";
import { CloseCircle, Move, Settings } from "@vicons/ionicons5";
import { useKeyboardStore } from "../../store/keyboard";

const props = defineProps<{
  index: number;
}>();

const keyboardStore = useKeyboardStore();

const store = useGlobalStore();
const elementRef = ref<HTMLElement | null>(null);

const isActive = computed(
  () => props.index === keyboardStore.activeButtonIndex
);
const keyMapping = computed(
  () => store.editKeyMappingList[props.index] as KeySteeringWheel
);

const offset = computed(() => {
  const keyboardElement = document.getElementById("keyboardElement");
  if (keyboardElement) {
    const clientWidth = keyboardElement.clientWidth;
    const screenSizeW =
      store.screenSizeW === 0 ? clientWidth : store.screenSizeW;
    return (keyMapping.value.offset * clientWidth) / screenSizeW;
  } else return keyMapping.value.offset;
});

function dragHandler(downEvent: MouseEvent) {
  keyboardStore.activeButtonIndex = props.index;
  keyboardStore.showButtonSettingFlag = false;
  const oldX = keyMapping.value.posX;
  const oldY = keyMapping.value.posY;
  const element = elementRef.value;
  if (element) {
    const keyboardElement = document.getElementById(
      "keyboardElement"
    ) as HTMLElement;
    const maxX = keyboardElement.clientWidth - offset.value;
    const maxY = keyboardElement.clientHeight - offset.value;

    const x = downEvent.clientX;
    const y = downEvent.clientY;
    const moveHandler = (moveEvent: MouseEvent) => {
      let newX = oldX + moveEvent.clientX - x;
      let newY = oldY + moveEvent.clientY - y;
      newX = Math.max(offset.value, Math.min(newX, maxX));
      newY = Math.max(offset.value, Math.min(newY, maxY));
      keyMapping.value.posX = newX;
      keyMapping.value.posY = newY;
    };
    window.addEventListener("mousemove", moveHandler);
    const upHandler = () => {
      window.removeEventListener("mousemove", moveHandler);
      window.removeEventListener("mouseup", upHandler);
      if (oldX !== keyMapping.value.posX || oldY !== keyMapping.value.posY) {
        keyboardStore.edited = true;
      }
    };
    window.addEventListener("mouseup", upHandler);
  }
}

function delCurKeyMapping() {
  keyboardStore.edited = true;
  keyboardStore.activeButtonIndex = -1;
  store.editKeyMappingList.splice(props.index, 1);
}

const settingPosX = ref(0);
const settingPosY = ref(0);
function showSetting() {
  const keyboardElement = document.getElementById(
    "keyboardElement"
  ) as HTMLElement;
  const maxWidth = keyboardElement.clientWidth - 179;
  const maxHeight = keyboardElement.clientHeight - 300;

  settingPosX.value = Math.min(
    keyMapping.value.posX + offset.value + 10,
    maxWidth
  );
  settingPosY.value = Math.min(keyMapping.value.posY - offset.value, maxHeight);
  keyboardStore.showButtonSettingFlag = true;
}
</script>

<template>
  <div
    :class="{ active: isActive }"
    :style="{
      left: `${keyMapping.posX - offset}px`,
      top: `${keyMapping.posY - offset}px`,
      width: `${offset * 2}px`,
      height: `${offset * 2}px`,
    }"
    @mousedown="dragHandler"
    class="key-steering-wheel"
    ref="elementRef"
  >
    <i />
    <span
      @mousedown="keyboardStore.activeSteeringWheelButtonKeyIndex = 0"
      :class="{
        'active-wheel':
          isActive && keyboardStore.activeSteeringWheelButtonKeyIndex == 0,
      }"
      >{{ keyMapping.key.up }}</span
    >
    <i />
    <span
      @mousedown="keyboardStore.activeSteeringWheelButtonKeyIndex = 2"
      :class="{
        'active-wheel':
          isActive && keyboardStore.activeSteeringWheelButtonKeyIndex == 2,
      }"
      >{{ keyMapping.key.left }}</span
    >
    <NIcon size="20">
      <Move />
    </NIcon>
    <span
      @mousedown="keyboardStore.activeSteeringWheelButtonKeyIndex = 3"
      :class="{
        'active-wheel':
          isActive && keyboardStore.activeSteeringWheelButtonKeyIndex == 3,
      }"
      >{{ keyMapping.key.right }}</span
    >
    <i />
    <span
      @mousedown="keyboardStore.activeSteeringWheelButtonKeyIndex = 1"
      :class="{
        'active-wheel':
          isActive && keyboardStore.activeSteeringWheelButtonKeyIndex == 1,
      }"
      >{{ keyMapping.key.down }}</span
    >
    <i />
    <NButton
      class="key-close-btn"
      text
      @click="delCurKeyMapping"
      :type="isActive ? 'primary' : 'info'"
      :style="{
        left: `${offset * 2 + 10}px`,
        bottom: `${offset * 2 - 20}px`,
      }"
    >
      <template #icon>
        <NIcon size="15">
          <CloseCircle />
        </NIcon>
      </template>
    </NButton>
    <NButton
      class="key-setting-btn"
      text
      @click="showSetting"
      :type="isActive ? 'primary' : 'info'"
      :style="{
        left: `${offset * 2 + 10}px`,
        top: `${offset * 2 - 20}px`,
      }"
    >
      <template #icon>
        <NIcon size="15">
          <Settings />
        </NIcon>
      </template>
    </NButton>
  </div>
  <div
    class="key-setting"
    v-if="isActive && keyboardStore.showButtonSettingFlag"
    :style="{
      left: `${settingPosX}px`,
      top: `${settingPosY}px`,
    }"
  >
    <NH4 prefix="bar">{{
      $t("pages.KeyBoard.SteeringWheel.steeringWheel")
    }}</NH4>
    <NFormItem :label="$t('pages.KeyBoard.SteeringWheel.offset')">
      <NInputNumber
        v-model:value="keyMapping.offset"
        :min="1"
        @update:value="keyboardStore.edited = true"
      />
    </NFormItem>
    <NFormItem :label="$t('pages.KeyBoard.setting.pointerID')">
      <NInputNumber
        v-model:value="keyMapping.pointerId"
        :min="0"
        :placeholder="$t('pages.KeyBoard.setting.pointerIDPlaceholder')"
        @update:value="keyboardStore.edited = true"
      />
    </NFormItem>
    <NFormItem :label="$t('pages.KeyBoard.setting.note')">
      <NInput
        v-model:value="keyMapping.note"
        :placeholder="$t('pages.KeyBoard.setting.notePlaceholder')"
        @update:value="keyboardStore.edited = true"
      />
    </NFormItem>
  </div>
</template>

<style scoped lang="scss">
.key-setting {
  position: absolute;
  display: flex;
  flex-direction: column;
  padding: 10px 20px;
  box-sizing: border-box;
  width: 170px;
  height: 300px;
  border-radius: 5px;
  border: 2px solid var(--light-color);
  background-color: var(--bg-color);
  z-index: 3;
}

.key-steering-wheel {
  position: absolute;
  border-radius: 50%;
  box-sizing: border-box;
  border: 2px solid var(--blue-color);
  display: flex;
  justify-content: center;
  align-items: center;
  font-size: 10px;
  font-weight: bold;

  display: grid;
  grid-template-columns: repeat(3, 33%);
  grid-template-rows: repeat(3, 33%);
  justify-items: center;
  align-items: center;

  &:not(.active):hover {
    border: 2px solid var(--light-color);
    color: var(--light-color);

    .n-icon {
      color: var(--light-color);
    }
  }

  span {
    cursor: pointer;
    &:hover {
      color: var(--primary-hover-color);
    }
  }

  .key-setting-btn,
  .key-close-btn {
    position: absolute;
  }
}

.active {
  border: 2px solid var(--primary-color);
  z-index: 2;

  .n-icon {
    color: var(--primary-color);
  }
}

.active-wheel {
  color: var(--primary-color);
}
</style>
