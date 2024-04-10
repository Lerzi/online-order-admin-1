<script setup>
import { apiAdd, apiUpdate, apiDetail } from '@/api/goods/spec'
import { useRoute } from 'vue-router';
const formRef = ref(null)
const form = ref({})


const submitLoading = ref(false)
async function submit() {
  try {
    submitLoading.value = true
    await formRef.value.validate()
    if (form.value.id) {
      await apiUpdate(form.value)
    } else {
      await apiAdd(form.value)
    }
    window.$message.success('保存成功')
    back()
  } catch (error) {
    console.error(error);
  } finally {
    submitLoading.value = false
  }
}

const isTakeout = computed({
  get() {
    if (!form.value.isTakeout) return []
    return form.value.isTakeout.split(',')
  },
  set(val) {
    if (!val) {
      form.value.isTakeout = ''
    } else {
      form.value.isTakeout = val.toString()
    }
  }

})

const router = useRouter()
function back() {
  router.back()
}
const route = useRoute()
onMounted(() => {
  if (route.params.id) {
    getDetail()
  }
})

async function getDetail() {
  try {
    const res = await apiDetail(route.params.id)
    form.value = res
  } catch (error) {
    console.error(error);
  }
}

</script>

<template>
  <n-card h-full min-w-lg>
    <n-page-header title="返回" @back="back()">
      <div min-w-lg flex flex-col items-center>
        <div w-full max-w-2xl>
          <n-form ref="formRef" :model="form" size="large">
            <n-form-item path="categoryName" label="规格名称"
              :rule="[{ required: true, min: 2, max: 5, message: '请输入2到5位的分类名称', trigger: 'blur' }]">
              <n-input v-model:value="form.categoryName" />
            </n-form-item>
            <n-form-item path="isShow" label="是否显示"
              :rule="[{ required: true, message: '请添加至少一个规格值', trigger: 'change' }]">
              <n-dynamic-tags v-model:value="specValues" />
            </n-form-item>

            <n-form-item path="icon" label="分类图标">
              <UploadImage v-model:value="form.icon"></UploadImage>
            </n-form-item>
          </n-form>
        </div>
        <div w-full max-w-2xl>
          <n-flex>
            <n-button type="primary" @click="submit" size="large" :loading="submitLoading">保存</n-button>
          </n-flex>
        </div>
      </div>
    </n-page-header>
  </n-card>
</template>
