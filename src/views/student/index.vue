<script setup lang="ts">
import { computed, ref } from 'vue';
import {
  NButton,
  NDatePicker,
  NForm,
  NFormItem,
  NGrid,
  NGridItem,
  NInput,
  NInputNumber,
  NModal,
  NSelect,
  useDialog,
  useMessage
} from 'naive-ui';
import type { FormInst, FormRules } from 'naive-ui';

interface Student {
  id: number;
  name: string;
  age: number;
  gender: string;
  grade: string;
  schoolBackground: string;
  serviceItems: string;
  teacher: string;
  leadSource: string;
  entryTime: number;
}

interface Filters {
  serviceItems: string | null;
  teacher: string | null;
  leadSource: string | null;
}

const students = ref<Student[]>([
  {
    id: 1,
    name: '张三',
    age: 18,
    gender: '男',
    grade: '高三',
    schoolBackground: '重点中学',
    serviceItems: '留学咨询',
    teacher: '王老师',
    leadSource: '官网',
    entryTime: Date.now()
  },
  {
    id: 2,
    name: '李四',
    age: 17,
    gender: '女',
    grade: '高二',
    schoolBackground: '实验中学',
    serviceItems: '升学规划',
    teacher: '张老师',
    leadSource: '朋友推荐',
    entryTime: Date.now()
  }
]);

const message = useMessage();
const dialog = useDialog();
const formRef = ref<FormInst | null>(null);
const showModal = ref(false);
const submitting = ref(false);
const editingId = ref<number | null>(null);

// 搜索和筛选相关
const searchText = ref('');
const filters = ref<Filters>({
  serviceItems: null,
  teacher: null,
  leadSource: null
});

const modalTitle = computed(() => (editingId.value === null ? '添加学生' : '编辑学生'));

// 对接老师选项（从现有数据中提取）
const teacherOptions = computed(() => {
  const teachers = new Set(students.value.map(s => s.teacher));
  return Array.from(teachers).map(teacher => ({
    label: teacher,
    value: teacher
  }));
});

// 服务项目选项
const serviceItemsOptions = [
  {
    label: '留学咨询',
    value: '留学咨询'
  },
  {
    label: '升学规划',
    value: '升学规划'
  },
  {
    label: '语言培训',
    value: '语言培训'
  },
  {
    label: '背景提升',
    value: '背景提升'
  }
];

// 筛选后的学生列表
const filteredStudents = computed(() => {
  return students.value.filter(student => {
    // 搜索文本过滤
    if (searchText.value) {
      const searchLower = searchText.value.toLowerCase();
      const matchSearch =
        student.name.toLowerCase().includes(searchLower) ||
        student.schoolBackground.toLowerCase().includes(searchLower);
      if (!matchSearch) return false;
    }

    // 服务项目过滤
    if (filters.value.serviceItems && student.serviceItems !== filters.value.serviceItems) {
      return false;
    }

    // 对接老师过滤
    if (filters.value.teacher && student.teacher !== filters.value.teacher) {
      return false;
    }

    // 线索来源过滤
    if (filters.value.leadSource && student.leadSource !== filters.value.leadSource) {
      return false;
    }

    return true;
  });
});

const formData = ref({
  name: '',
  age: null as number | null,
  gender: null as string | null,
  grade: null as string | null,
  schoolBackground: '',
  serviceItems: '',
  teacher: '',
  leadSource: null as string | null,
  entryTime: null as number | null
});

const rules: FormRules = {
  name: {
    required: true,
    message: '请输入姓名',
    trigger: ['blur', 'input']
  },
  age: {
    required: true,
    message: '请输入年龄',
    trigger: ['blur', 'change']
  },
  gender: {
    required: true,
    message: '请选择性别',
    trigger: ['blur', 'change']
  },
  grade: {
    required: true,
    message: '请选择年级',
    trigger: ['blur', 'change']
  },
  schoolBackground: {
    required: true,
    message: '请输入院校背景',
    trigger: ['blur', 'input']
  },
  serviceItems: {
    required: true,
    message: '请输入服务项目',
    trigger: ['blur', 'input']
  },
  teacher: {
    required: true,
    message: '请输入对接老师',
    trigger: ['blur', 'input']
  },
  leadSource: {
    required: true,
    message: '请选择线索来源',
    trigger: ['blur', 'change']
  },
  entryTime: {
    required: true,
    message: '请选择接入时间',
    trigger: ['blur', 'change']
  }
};

const genderOptions = [
  {
    label: '男',
    value: '男'
  },
  {
    label: '女',
    value: '女'
  }
];

const gradeOptions = [
  {
    label: '高一',
    value: '高一'
  },
  {
    label: '高二',
    value: '高二'
  },
  {
    label: '高三',
    value: '高三'
  }
];

const leadSourceOptions = [
  {
    label: '官网',
    value: '官网'
  },
  {
    label: '朋友推荐',
    value: '朋友推荐'
  },
  {
    label: '社交媒体',
    value: '社交媒体'
  },
  {
    label: '线下活动',
    value: '线下活动'
  },
  {
    label: '其他',
    value: '其他'
  }
];

function handleAdd() {
  editingId.value = null;
  resetForm();
  showModal.value = true;
}

async function handleSubmitForm() {
  if (!formRef.value) return;

  try {
    submitting.value = true;
    await formRef.value.validate();

    const studentData = {
      name: formData.value.name,
      age: formData.value.age!,
      gender: formData.value.gender!,
      grade: formData.value.grade!,
      schoolBackground: formData.value.schoolBackground,
      serviceItems: formData.value.serviceItems,
      teacher: formData.value.teacher,
      leadSource: formData.value.leadSource!,
      entryTime: formData.value.entryTime!
    };

    if (editingId.value === null) {
      // 添加新学生
      const newId = Math.max(...students.value.map(s => s.id)) + 1;
      students.value.push({
        id: newId,
        ...studentData
      });
      message.success('添加成功');
    } else {
      // 更新学生信息
      const index = students.value.findIndex(s => s.id === editingId.value);
      if (index !== -1) {
        students.value[index] = {
          id: editingId.value,
          ...studentData
        };
        message.success('更新成功');
      }
    }

    showModal.value = false;
    resetForm();
  } catch {
    // 单验证失���
  } finally {
    submitting.value = false;
  }
}

function resetForm() {
  formData.value = {
    name: '',
    age: null,
    gender: null,
    grade: null,
    schoolBackground: '',
    serviceItems: '',
    teacher: '',
    leadSource: null,
    entryTime: null
  };
  if (formRef.value) {
    formRef.value.restoreValidation();
  }
}

function handleEdit(row: Student) {
  editingId.value = row.id;
  formData.value = {
    name: row.name,
    age: row.age,
    gender: row.gender,
    grade: row.grade,
    schoolBackground: row.schoolBackground,
    serviceItems: row.serviceItems,
    teacher: row.teacher,
    leadSource: row.leadSource,
    entryTime: row.entryTime
  };
  showModal.value = true;
}

function handleDelete(row: Student) {
  dialog.warning({
    title: '确认删除',
    content: `确定要删除学生 ${row.name} 吗？`,
    positiveText: '确定',
    negativeText: '取消',
    onPositiveClick: () => {
      const index = students.value.findIndex(s => s.id === row.id);
      if (index !== -1) {
        students.value.splice(index, 1);
        message.success('删除成功');
      }
    }
  });
}
</script>

<template>
  <div class="page-container">
    <div class="header-actions">
      <div class="left">
        <h2 class="title">学生管理 ({{ filteredStudents.length }})</h2>
      </div>
      <div class="right">
        <NButton type="primary" class="add-button" @click="handleAdd">
          <template #icon>
            <div class="i-carbon:add" />
          </template>
          添加学生
        </NButton>
      </div>
    </div>

    <div class="search-filters">
      <NInput v-model:value="searchText" type="text" placeholder="搜索姓名、院校背景..." clearable class="search-input">
        <template #prefix>
          <div class="i-carbon:search" />
        </template>
      </NInput>
      <NSelect
        v-model:value="filters.serviceItems"
        placeholder="服务项目"
        clearable
        :options="serviceItemsOptions"
        class="filter-item"
      />
      <NSelect
        v-model:value="filters.teacher"
        placeholder="对接老师"
        clearable
        :options="teacherOptions"
        class="filter-item"
      />
      <NSelect
        v-model:value="filters.leadSource"
        placeholder="线索来源"
        clearable
        :options="leadSourceOptions"
        class="filter-item"
      />
    </div>

    <div class="student-list">
      <div v-for="student in filteredStudents" :key="student.id" class="student-card">
        <div class="card-header">
          <div class="student-info">
            <div class="avatar">{{ student.name.charAt(0) }}</div>
            <div class="basic-info">
              <h3>{{ student.name }}</h3>
              <p class="subtitle">{{ student.schoolBackground }}</p>
            </div>
          </div>
          <div class="actions">
            <NButton quaternary circle type="primary" @click="handleEdit(student)">
              <template #icon>
                <div class="i-carbon:edit" />
              </template>
            </NButton>
            <NButton quaternary circle type="error" @click="handleDelete(student)">
              <template #icon>
                <div class="i-carbon:trash-can" />
              </template>
            </NButton>
          </div>
        </div>
        <div class="card-content">
          <div class="info-grid">
            <div class="info-item">
              <label>年龄</label>
              <span>{{ student.age }}岁</span>
            </div>
            <div class="info-item">
              <label>性别</label>
              <span>{{ student.gender }}</span>
            </div>
            <div class="info-item">
              <label>年级</label>
              <span>{{ student.grade }}</span>
            </div>
            <div class="info-item">
              <label>服务项目</label>
              <span>{{ student.serviceItems }}</span>
            </div>
            <div class="info-item">
              <label>对接老师</label>
              <span>{{ student.teacher }}</span>
            </div>
            <div class="info-item">
              <label>线索来源</label>
              <span>{{ student.leadSource }}</span>
            </div>
          </div>
          <div class="entry-time">接入时间：{{ new Date(student.entryTime).toLocaleDateString() }}</div>
        </div>
      </div>
    </div>

    <NModal v-model:show="showModal" preset="card" :style="{ width: '600px' }" :title="modalTitle">
      <NForm
        ref="formRef"
        :model="formData"
        :rules="rules"
        label-placement="left"
        label-width="100"
        require-mark-placement="right-hanging"
        size="medium"
      >
        <NGrid :cols="2" :x-gap="24">
          <NGridItem>
            <NFormItem label="姓名" path="name">
              <NInput v-model:value="formData.name" placeholder="请输入姓名" />
            </NFormItem>
          </NGridItem>
          <NGridItem>
            <NFormItem label="年龄" path="age">
              <NInputNumber v-model:value="formData.age" placeholder="请输入年龄" :min="1" :max="100" />
            </NFormItem>
          </NGridItem>
          <NGridItem>
            <NFormItem label="性别" path="gender">
              <NSelect v-model:value="formData.gender" :options="genderOptions" placeholder="请选择性别" />
            </NFormItem>
          </NGridItem>
          <NGridItem>
            <NFormItem label="年级" path="grade">
              <NSelect v-model:value="formData.grade" :options="gradeOptions" placeholder="请选择年级" />
            </NFormItem>
          </NGridItem>
        </NGrid>
        <NFormItem label="院校背景" path="schoolBackground">
          <NInput v-model:value="formData.schoolBackground" placeholder="请输入院校背景" />
        </NFormItem>
        <NFormItem label="服务项目" path="serviceItems">
          <NInput v-model:value="formData.serviceItems" placeholder="请输入服务项目" />
        </NFormItem>
        <NFormItem label="对接老师" path="teacher">
          <NInput v-model:value="formData.teacher" placeholder="请输入对接老师" />
        </NFormItem>
        <NFormItem label="线索来源" path="leadSource">
          <NSelect v-model:value="formData.leadSource" :options="leadSourceOptions" placeholder="请选择线索来源" />
        </NFormItem>
        <NFormItem label="接入时间" path="entryTime">
          <NDatePicker v-model:value="formData.entryTime" type="date" placeholder="请选择接入时间" />
        </NFormItem>
      </NForm>
      <template #footer>
        <div class="modal-footer">
          <NButton type="default" @click="showModal = false">取消</NButton>
          <NButton type="primary" :loading="submitting" @click="handleSubmitForm">确定</NButton>
        </div>
      </template>
    </NModal>
  </div>
</template>

<style scoped>
.page-container {
  padding: 24px;
  background-color: #f5f7fa;
  min-height: 100vh;
}

.header-actions {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 24px;
}

.title {
  font-size: 24px;
  font-weight: 500;
  color: #1f2937;
  margin: 0;
}

.add-button {
  font-weight: 500;
  padding: 8px 20px;
}

.student-list {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(400px, 1fr));
  gap: 24px;
}

.student-card {
  background: white;
  border-radius: 16px;
  padding: 20px;
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.1);
  transition: all 0.3s ease;
}

.student-card:hover {
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
}

.card-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 20px;
}

.student-info {
  display: flex;
  align-items: center;
  gap: 16px;
}

.avatar {
  width: 48px;
  height: 48px;
  border-radius: 50%;
  background: #4f46e5;
  color: white;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 20px;
  font-weight: 500;
}

.basic-info h3 {
  margin: 0;
  font-size: 18px;
  font-weight: 500;
  color: #1f2937;
}

.subtitle {
  margin: 4px 0 0;
  color: #6b7280;
  font-size: 14px;
}

.actions {
  display: flex;
  gap: 8px;
}

.info-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 16px;
  margin-bottom: 16px;
}

.info-item {
  display: flex;
  flex-direction: column;
  gap: 4px;
}

.info-item label {
  font-size: 13px;
  color: #6b7280;
}

.info-item span {
  font-size: 14px;
  color: #1f2937;
}

.entry-time {
  font-size: 13px;
  color: #6b7280;
  padding-top: 12px;
  border-top: 1px solid #e5e7eb;
}

.modal-footer {
  display: flex;
  justify-content: flex-end;
  gap: 12px;
  margin-top: 24px;
}

:deep(.n-modal) {
  border-radius: 16px;
}

:deep(.n-card) {
  border-radius: 16px;
}

:deep(.n-button) {
  border-radius: 8px;
}

/* 添加搜索和筛选相关样式 */
.search-filters {
  margin-bottom: 24px;
  display: flex;
  gap: 16px;
  align-items: center;
}

.search-input {
  width: 300px;
}

.filter-item {
  width: 160px;
}
</style>
