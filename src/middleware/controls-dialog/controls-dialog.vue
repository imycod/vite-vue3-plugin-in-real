<template>
  <el-dialog v-model="state.visible" :title="state.title" :before-close="handleClose">
    <component :is="componentName" ref="componentRef" v-if="isVisible" :data="componentData"></component>
    <template #footer>
      <span class="dialog-footer">
        <el-button @click="handleClose">Cancel</el-button>
        <el-button type="primary" @click="handleSubmit"> Confirm </el-button>
      </span>
    </template>
  </el-dialog>
</template>

<script setup lang="ts">
import { ref, nextTick, shallowRef } from "vue";
import useDialog from "@/hooks/useDialog.ts";

const { setState: setDialogState, state, close } = useDialog();

const isVisible = ref(false)

const componentName = ref();
const componentRef = ref(null);
const componentData = ref(null);

// let initialState = {}
let stateCache = {}
// 接口供外部实现
let callback = () => { }
const setState = (state) => {
  // 注意引用不可随意更改，否则深克隆
  stateCache = state

  componentName.value = state.componentName;
  componentData.value = state.data

  if (state.callback) {
    callback = state.callback;
  }

  isVisible.value = state.visible;

  setDialogState({ title: state.title, visible: state.visible });

  nextTick(() => {
    if (state.data) {
      componentRef.value.initState && componentRef.value.initState(state.data);
    }
  })
};
const handleSubmit = () => {
  try {
    // componentRef.value.handleSubmit((done = true) => {
    //   if (done) {
    //     close();
    //   }
    // });
    if (componentRef.value && componentRef.value.handleSubmit) {
      componentRef.value.handleSubmit(async () => {
        try {
          const done = await callback();
          if (done) {
            close();
          }
        } catch (error) {
          alert(error.message)
        }
      }, { ...stateCache, close });
    } else {
      callback(close);
    }
  } catch (e) {
    alert('出错了')
    console.log(e.message);
  }
};

function handleClose() {
  close()
  setTimeout(() => {
    isVisible.value = false
  }, 200)
}
defineExpose({
  setState,
});
</script>