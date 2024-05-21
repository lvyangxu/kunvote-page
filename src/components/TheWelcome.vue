<script setup lang="ts">
import { watch } from 'vue'
import { contractInstance, accounts } from '../util/contract'
import { ref } from 'vue'

async function vote(option: number) {
  if (voted.value) {
    return
  }

  // 调用合约的一个函数（例如 getBalance）
  try {
    await contractInstance.methods.vote(option).send({ from: currentAccount.value as any })
    await calc()
  } catch (error) {
    console.log(error)
  }
}

async function calc() {
  try {
    const data: { '0': boolean; '1': bigint[] } = await contractInstance.methods
      .calc()
      .call({ from: currentAccount.value as any })
    stars.value = data[1].map((d) => Number(d))
    voted.value = data[0]
  } catch (error) {
    console.log(error)
  }
}
const data = [
  { img: '/xiaoheizi.jpeg', title: '小黑子' },
  { img: '/chunluren.png', title: '纯鹿人' },
  { img: '/zhenikun.png', title: '真ikun' }
]

const stars = ref([0, 0, 0])
const voted = ref(false)
const selectData = accounts.map((d, i) => {
  return { title: `测试账号${i + 1}`, value: d }
})
const currentAccount = ref<(typeof selectData)[0] | null>(null)

watch(currentAccount, () => {
  calc()
})
</script>

<template>
  <div v-if="currentAccount === null">
    <h2>选择一个测试账号进行模拟</h2>

    <v-select
      :style="{ width: '300px', background: 'none' }"
      density="compact"
      v-model="currentAccount"
      :items="selectData"
      item-title="title"
      item-value="value"
    >
    </v-select>
  </div>

  <v-card
    v-if="currentAccount !== null"
    width="1200px"
    :style="{ margin: 'auto', height: '80vh', marginTop: '10vh' }"
  >
    <div>
      <h2 :style="{ textAlign: 'center' }">{{ voted ? '你已经投过票了' : '请选择你的阵营' }}</h2>
      <div
        :style="{
          gap: '5px',
          display: 'flex',
          flexDirection: 'row',
          justifyContent: 'space-around',
          padding: '20px'
        }"
      >
        <div v-for="(item, index) in data" :key="index">
          <v-avatar size="200">
            <img :src="item.img" :style="{ width: 200 + 'px' }" />
          </v-avatar>
          <h3 :style="{ textAlign: 'center' }">{{ item.title }}</h3>
          <div
            :style="{
              display: 'flex',
              alignItems: 'center',
              justifyContent: 'center',
              gap: '10px'
            }"
          >
            <v-icon
              :icon="voted ? 'mdi-heart' : 'mdi-heart-outline'"
              :style="voted ? {} : { color: 'red', cursor: 'pointer' }"
              @click="vote(index + 1)"
            />
            <div>{{ stars[index] }}</div>
          </div>
        </div>
      </div>
    </div>
  </v-card>
</template>
