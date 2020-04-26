<template>
  <div class="flex flex-col evaluation">
    <h3>展讲余额计算系统 —— 离线版 v1.0</h3>

    <!-- 操作按钮栏 -->
    <div class="flex jc-center actions">
      <button
        class="action-button shadow"
        :class="[
          (fakeRoute === 'init' ? 'focus-green' : void 0)
        ]"
        @click="fakeRouterPush(fakeRoute === 'init' ? '' : 'init')"
      >{{ fakeRoute === 'init' ? "关闭初始化组" : "初始化组" }}</button>
      <button
        class="action-button shadow"
        :class="[
          (fakeRoute === 'evaluation' ? 'focus-green' : void 0)
        ]"
        @click="fakeRouterPush(fakeRoute === 'evaluation' ? '' : 'evaluation')"
      >{{ fakeRoute === 'evaluation' ? "关闭评分页" : "进入评分" }}</button>
    </div>

    <!-- 操作对应界面 -->
    <div v-show="fakeRoute === 'init'" class="flex flex-col jc-start new-group-view tb-gap">
      <h4>初始化组</h4>
      <!-- 组员数量输入框 -->
      <div class="flex form-item">
        <label class="bold" for="memberCount">组员数量：</label>
        <input
          class="shadow com-inputer"
          name="memberCount"
          placeholder="请输入组员数量"
          v-model="newGroupForm.memberCount"
          type="text"
        />
      </div>

      <ul
        v-show="newGroupForm.memberCount !== ''"
        class="initial-list no-list-style shadow flex flex-col"
      >
        <li class="initial-list-item flex" v-for="(e, i) in newGroupForm.memberList" :key="i">
          <label class="bold">姓名：</label>
          <input
            @input="arrayElementAssignFromEvent(newGroupForm.memberList, i, {
              name: $event.target.value
            })"
            class="com-inputer shadow"
            type="text"
            placeholder="请输入组员姓名"
          />
          <button @click="getInitScore(i)" class="action-button shadow">初始化得分</button>
          <label class="final-init-score bold">初始余额: {{ e.balance || '' }}</label>
        </li>
      </ul>

      <button
        v-show="newGroupForm.memberList.length !== 0"
        class="shadow action-button save-init-info"
      >保存初始化信息</button>
    </div>

    <!-- 展讲评分页面 -->
    <p v-show="alertShow" class="alert-info">
      {{ alertInfo }}
      <span>提示信息 2s 后消失</span>
    </p>
    <div
      v-show="fakeRoute === 'evaluation'"
      class="speech-evaluation-view tb-gap flex flex-col jc-start"
    >
      <div class="form-item flex flex-col">
        <div class="flex">
          <label class="bold" for="speaker">展讲者：</label>
          <input
            v-model="evaluationForm.speakerName"
            name="speaker"
            type="text"
            class="shadow com-inputer"
            placeholder="请输入展讲者姓名"
          />
        </div>

        <div class="flex tb-gap">
          <label class="bold" for="audienceCount">听众数量：</label>
          <input
            v-model.number="evaluationForm.audienceCount"
            name="audienceCount"
            type="text"
            class="shadow com-inputer"
            placeholder="请输入听众数量"
          />
        </div>

        <div class="flex tb-gap">
          <button @click="speakerGotCompute" class="action-button shadow lr-gap">执行计算</button>
          <label class="bold">最终得分：</label>
          <div>{{ evaluationForm.speakerGot || '等待计算...' }}</div>
        </div>
      </div>

      <ul class="tb-gap audience-list no-list-style flex flex-col">
        <li class="audience-list-item flex" v-for="(e, i) in evaluationForm.audienceList" :key="i">
          <input
            @input="arrayElementAssignFromEvent(evaluationForm.audienceList, i, {
              name: $event.target.value
            })"
            type="text"
            class="com-inputer shadow"
            placeholder="请输入听众姓名"
          />

          <label class="bold">{{ e.name === '' ? `听众 ${i+1}` : e.name }} 原余额：</label>
          <input
            @input="arrayElementAssignFromEvent(evaluationForm.audienceList, i, {
              beforeBalance: parseInt($event.target.value)
            })"
            type="text"
            class="com-inputer shadow"
            placeholder="请输入听众原余额"
          />

          <label class="bold">打分：</label>
          <input
            @input="arrayElementAssignFromEvent(evaluationForm.audienceList, i, {
              give: parseInt($event.target.value)
            })"
            type="number"
            class="com-inputer shadow short"
            placeholder="输入给分"
          />

          <button @click="supplementCompute(i)" class="shadow action-button">补给计算</button>
          <label v-show="e.supplement" class="bold">补给： {{ e.supplement }} 最终余额：{{ e.afterBalance }}</label>
        </li>
      </ul>
    </div>
  </div>
</template>

<script>
// 工具函数
function RollTheDice() {
  return Math.round(Math.random() * 5) + 1;
}
function initNewGroupForm() {
  return {
    memberCount: '',
    memberList: []
  };
}
function initEvaluationForm() {
  return {
    speakerName: '',
    speakerGot: NaN,
    audienceCount: null,
    audienceList: []
  };
}

export default {
  name: 'Evaluation',
  data() {
    return {
      // 自制路由控制器
      fakeRoute: 'evaluation',
      alertInfo: '',
      alertShow: false,

      // 初始化一组时需要用到的数据
      newGroupForm: initNewGroupForm(),
      // 评分时需要用到的数据
      evaluationForm: initEvaluationForm()
    };
  },
  watch: {
    'newGroupForm.memberCount': function() {
      this.buildList(
        this.newGroupForm.memberCount,
        this.newGroupForm.memberList,
        {
          name: '',
          balance: NaN // 保持结果为 数字类型
        }
      );
    },
    'evaluationForm.audienceCount': function() {
      this.buildList(
        this.evaluationForm.audienceCount,
        this.evaluationForm.audienceList,
        {
          name: '',
          beforeBalance: NaN,
          give: NaN,
          supplement: NaN,
          afterBalance: NaN
        }
      );
    },
    'evaluationForm.audienceList': function() {}
  },
  methods: {
    fakeRouterPush(route) {
      this.fakeRoute = route;
      window.location.hash = `/${route}`;

      switch (route) {
        case 'init':
          this.newGroupForm = initNewGroupForm();
          break;
        case 'evaluation':
          this.evaluationForm = initEvaluationForm();
          break;
      }
    },
    assignHack(target, key, value) {
      return Object.assign({}, target[key], value);
    },
    assignSet(target, key, value) {
      this.$set(target, key, this.assignHack(target, key, value));
    },
    // 构建列表
    buildList(countStr, list, initStructure) {
      list.splice(0, list.length);
      for (let i = 0; i < parseInt(countStr); i++) {
        list.push(initStructure);
      }
    },
    // 初始化得分
    getInitScore(i) {
      this.assignSet(this.newGroupForm.memberList, i, {
        balance: 140 + RollTheDice() * Math.round(10 * (0.5 + Math.random()))
      });
    },
    arrayElementAssignFromEvent(arr, index, src) {
      this.$set(arr, index, this.assignHack(arr, index, src));
    },
    // 计算最终得分：
    speakerGotCompute() {
      let hasEmpty = false;
      this.evaluationForm.audienceList.forEach(e => {
        if (isNaN(e.give)) hasEmpty = true;
      });
      if (!this.evaluationForm.audienceCount || hasEmpty) {
        this.alertInfo = '请先完整录入给分！';
        this.alertShow = true;
        setTimeout(() => {
          this.alertShow = false;
        }, 2200);
      }

      const sortedMinToMax = this.evaluationForm.audienceList.sort((a, b) => {
        return a.give - b.give;
      });
      const allLength = sortedMinToMax.length;
      const minAndMaxAverage = Math.round(
        (sortedMinToMax[0].give +
          sortedMinToMax[sortedMinToMax.length - 1].give) /
          2
      );
      let otherSum = 0;
      sortedMinToMax.forEach((e, i) => {
        if (i === 0 || i === allLength - 1) return;
        otherSum += e.give;
      });
      this.evaluationForm.speakerGot = Math.round(
        (otherSum + minAndMaxAverage) / (allLength - 1)
      );
    },
    // 计算补给：余额补给的公式：损失比：给分与最终分作比（大数做分母） ×（ 100 + （给分与最终分的差））
    supplementCompute(i) {
      if (!this.evaluationForm.speakerGot) {
        this.alertInfo = '请先执行 "最终得分" 的计算！';
        this.alertShow = true;
        setTimeout(() => {
          this.alertShow = false;
        }, 2200);
        return;
      }

      const it = this.evaluationForm.audienceList[i];
      const tempRate = it.give / this.evaluationForm.speakerGot;
      let looseRate = tempRate > 1 ? 1 / tempRate : tempRate;
      const supplement = Math.round(
        looseRate * (100 + Math.abs(this.evaluationForm.speakerGot - it.give))
      );

      this.assignSet(this.evaluationForm.audienceList, i, {
        supplement,
        afterBalance: it.beforeBalance - it.give + supplement
      });
    }
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="scss">
.shadow {
  border-radius: 25px;
  box-shadow: 0 5px 10px rgba(0, 0, 0, 0.15);
}
.short {
  width: 100px;
}

.action-button {
  min-width: 80px;
  padding: 8px 20px;
  outline: none;
  border: none;
  cursor: pointer;
  font-size: 16px;
  font-weight: bold;
  transition: all 0.4s ease;
}
.focus-green {
  color: #0a8;
}

.com-inputer {
  border: none;
  padding: 10px 20px;

  &:focus {
    outline: none;
  }
}

.evaluation {
  h1,
  h2,
  h3,
  h4,
  h5 {
    margin: 5px auto;
  }

  .actions {
    padding: 0 0 14px 0;
    border-bottom: 1px solid rgba(0, 0, 0, 0.15);
    * {
      margin: 5px 10px;
    }
  }

  .new-group-view {
    width: max-content;
    min-width: 800px;
    padding: 20px 10px 50px 10px;

    .initial-list {
      padding: 10px;
      min-width: max-content;
      max-width: 60%;
      margin: 40px auto 20px auto;

      &-item {
        padding: 10px;

        * {
          margin: 5px 10px;
        }
        .final-init-score {
          color: green;
        }
      }
    }

    .save-init-info {
      margin-top: 20px;
    }
  }

  .alert-info {
    padding: 10px;
    border-left: 6px solid rgba(80, 76, 76, 0.4);
    background-color: #ab450a5e;
    color: #423a3a;
    font-size: 18px;
    font-weight: bold;

    span {
      font-weight: 500;
      font-size: 12px;
    }
  }

  .speech-evaluation-view {
    .audience-list {
      li {
        * {
          margin: 5px 10px;
        }
      }
    }
  }
}
</style>
