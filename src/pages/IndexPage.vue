<template>
  <q-page class="q-ma-xl">
    <span class="text-h3 row q-my-md">開放中文轉換 - OpenCC</span>
    <a href="https://github.com/BYVoid/OpenCC" target="_blank">關於 OpenCC</a>

    <div class="row q-mt-xl">
      <q-card class="col">
        <q-card-section>
          <div class="text-h6 text-bold">參數</div>
        </q-card-section>

        <q-card-section>
          <q-markup-table flat>
            <tbody>
              <tr>
                <td class="text-left">{{ options.from.label }}</td>
                <td class="text-left">
                  <q-radio
                    v-for="item in options.from.options"
                    :key="item.val"
                    v-model="from"
                    :label="item.label"
                    :val="item.val"
                  />
                </td>
              </tr>

              <tr>
                <td class="text-left">{{ options.to.label }}</td>
                <td class="text-left">
                  <q-radio
                    v-for="item in filteredTargets"
                    :key="item.val"
                    v-model="to"
                    :label="item.label"
                    :val="item.val"
                  />
                </td>
              </tr>

              <tr>
                <td class="text-left">{{ options.varient.label }}</td>
                <td class="text-left">
                  <q-radio
                    v-for="item in filteredVarients"
                    :key="item.val"
                    v-model="varient"
                    :label="item.label"
                    :val="item.val"
                  />
                </td>
              </tr>

              <tr>
                <td class="text-left">{{ options.region.label }}</td>
                <td class="text-left">
                  <q-radio
                    v-for="item in filteredRegions"
                    :key="item.val"
                    v-model="region"
                    :label="item.label"
                    :val="item.val"
                  />
                </td>
              </tr>

              <tr>
                <td class="text-left">設定檔</td>
                <td class="text-left">
                  {{ confStr }}
                </td>
              </tr>
            </tbody>
          </q-markup-table>
        </q-card-section>

        <q-card-section>
          <q-input
            v-model="text"
            autogrow
            outlined
            style="max-height: 50vh; overflow: scroll"
            type="textarea"
          />
        </q-card-section>

        <q-card-section class="row justify-center">
          <q-btn
            color="primary"
            label="轉換"
            :loading="loading"
            @click="convert"
          />
        </q-card-section>
      </q-card>
    </div>
  </q-page>
</template>

<script setup lang="ts">
import { ref, computed, watch } from 'vue';

const from = ref('s');
const to = ref('t');
const varient = ref('t');
const region = ref('none');
const text = ref('');
const loading = ref(false);

const options = {
  from: {
    label: '原文',
    options: [
      { val: 's', label: '簡體中文' },
      { val: 't', label: '繁體中文' },
      { val: 'jp', label: '日文新字體' },
    ],
  },
  to: {
    label: '目標',
    options: [
      { val: 's', label: '簡體中文' },
      { val: 't', label: '繁體中文' },
      { val: 'jp', label: '日文新字體' },
    ],
  },
  varient: {
    label: '異體字轉換',
    options: [
      { val: 't', label: 'OpenCC 標準' },
      { val: 'hk', label: '香港標準' },
      { val: 'tw', label: '臺灣標準' },
    ],
  },
  region: {
    label: '地域用詞轉換',
    options: [
      { val: 'none', label: '不轉換' },
      { val: 'sp', label: '中國大陸模式' },
      { val: 'twp', label: '臺灣模式' },
    ],
  },
};

const optsTree = {
  s: {
    t: {
      t: { none: 's2t.json' },
      hk: { none: 's2hk.json' },
      tw: { none: 's2tw.json', twp: 's2twp.json' },
    },
  },
  t: {
    s: {
      t: { none: 't2s.json', sp: 'tw2sp.json' },
    },
    t: {
      t: { none: 'tw2t.json' },
      hk: { none: 't2hk.json' },
      tw: { none: 't2tw.json', twp: 't2twp.json' },
    },
    jp: {
      t: { none: 'tw2jp.json' },
    },
  },
  jp: {
    t: { t: { none: 'jp2t.json' } },
  },
};

// computed
const confStr = computed(() => {
  return optsTree[from.value][to.value][varient.value][region.value];
});

const filteredTargets = computed(() =>
  options.from.options.filter((el) =>
    Object.keys(optsTree[from.value]).includes(el.val)
  )
);

const filteredVarients = computed(() => {
  const varientTree = optsTree[from.value][to.value];
  const keys = Object.keys(varientTree);
  return options.varient.options.filter((el) => keys.includes(el.val));
});

const filteredRegions = computed(() => {
  const regionTree = optsTree[from.value][to.value][varient.value];
  const keys = Object.keys(regionTree);
  return options.region.options.filter((el) => keys.includes(el.val));
});

async function convert() {
  try {
    loading.value = true;

    const resp = await fetch('/api/opencc', {
      method: 'POST',
      cache: 'no-cache',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ text: text.value, config: confStr.value }),
    });
    const data = await resp.json();
    text.value = data.text;
  } finally {
    loading.value = false;
  }
}

// watches
watch(from, () => {
  to.value = filteredTargets.value[0].val;
  varient.value = filteredVarients.value[0].val;
  region.value = filteredRegions.value[0].val;
});

watch(to, () => {
  varient.value = filteredVarients.value[0].val;
  region.value = filteredRegions.value[0].val;
});

watch(varient, () => {
  region.value = filteredRegions.value[0].val;
});
</script>
