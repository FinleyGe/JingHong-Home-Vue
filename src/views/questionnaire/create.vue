<template>
  <div class="container">
    <div class="add-bar">
      <div class="problem-add" @click="addIssue('Single')">
        + 创建单选题
      </div>
      <div class="problem-add" @click="addIssue('Multiple')">
        + 创建多选题
      </div>
      <div class="problem-add" @click="addIssue('Subjective')">
        + 创建填空题
      </div>
    </div>
    <div class="question-bar">
      <div class="title">
        <input placeholder="输入标题(不能为空)" v-model="title">
      </div>
      <hr>
      <div v-for="(item, index) in questions" class="issue-container" :key="index">
        <issueCard :type="item.type" :data="item" :index="index" @update="onUpdated" @deleteIssue="deleteIssue">
        </issueCard>
      </div>
      <div class="buttom-container">
        <j-h-button v-if="isNew" type="middle" @click="saveDraft()">保存为草稿</j-h-button>
        <j-h-button v-else @click="updateQ()" type="middle">提交</j-h-button>
      </div>
    </div>
  </div>
</template>

<script lang="ts" setup>
import JHButton from "@/components/JHButton.vue";
import { onMounted, ref } from "vue";
import router from "@/router";
import IssueCard from "@/components/IssueCard.vue";
import { getQuestionnaireById, updateQuestionnaire, createQuestionnaire } from "@/apis/questionnaire";
import { useQuestionnaireStore } from "@/stores/questionnaire";
const data = ref();
const questions = ref([]);
const title = ref();
const isNew = ref();
const pinia = useQuestionnaireStore();
onMounted(() => {
  console.log('create');
  console.log(pinia.selectedId);
  if (pinia.getIsAdmin() !== "true")
    router.push('/questionnaire/user');
  if (pinia.selectedId !== -1) {
    isNew.value = false;
    getQuestionnaireById(pinia.selectedId.toString()).then(res => {
      if (res.msg === 'ok') {
        console.log(res.data);
        data.value = res.data;
        title.value = data.value.title;
        questions.value = data.value.list;
      }
      else {
        alert('请求错误');
      }
    })
  }
  else {
    isNew.value = true;
    title.value = '';
    questions.value = [];
    data.value = {
      title: '',
      questions: []
    }
  }
})
function onUpdated(updateData, index) {
  questions.value[index] = updateData;
}
function deleteIssue(index) {
  questions.value.splice(index, 1);
}
function addIssue(type) {
  const issue = {
    type: type,
    text: '',
    options: [
      '',
      '',
      '',
    ]
  }
  questions.value.push(issue);
}
function saveDraft() {

  let data = {
    title: title.value,
    list: questions.value,
  }
  console.log('最终发送的数据：')
  console.log(data);
  createQuestionnaire(data).then(res => {
    if (res.msg === 'ok') {
      alert('保存成功');
    }
    else {
      alert('保存失败');
    }
  })
}

function updateQ() {
  let list = [];
  questions.value.forEach(item => {
    let question = {
      type: item.type,
      text: item.text,
      options: item.options
    }
    list.push(question);
  })
  let postData = {
    id: pinia.selectedId,
    list: list,
  }
  console.log("最终发送的数据：")
  console.log(postData);
  updateQuestionnaire(postData).then(res => {
    if (res.msg === 'ok') {
      alert('更新成功');
    }
    else {
      alert('更新失败');
    }
  })
}

</script>

<style scoped lang="scss">
.container {
  margin-top: 100px;
  background-color: rgb(239, 239, 239);
  display: flex;
  flex-direction: row;
  justify-content: space-around;
}

.add-bar {
  background-color: white;
  display: flex;
  flex-direction: column;
  justify-content: space-around;
  align-items: center;
  border-radius: 20px;
}

.problem-add {
  background-color: rgb(239, 239, 239);
  padding-inline: 20px;
  padding-block: 5px;
  margin-block: 5px;
  margin-inline: 10px;
  display: flex;
  justify-content: center;
  align-items: center;
  border-radius: 20px;
}

.question-bar {
  background-color: white;
  flex-direction: column;
  border-radius: 20px;
  max-height: 100vh;
  overflow: auto;
  padding: 20px;
}

.title {
  display: flex;
  align-items: center;
  border-radius: 20px;
  display: flex;
  font-size: 24px;
}

.title input {
  font-size: 20px;
  border-radius: 20px;
  background-color: rgb(239, 239, 239);
}

.issue-container {
  display: block;
}

hr {
  width: 80%;
  height: 2px;
  background-color: black;
}

.buttom-container {
  display: flex;
  justify-content: space-around;
}
</style>
