<template>
  <div class="flex flex-col gap-4">
    <ProjectPageSettingsBlock
      background
      title="Project Info"
      :disabled-message="disabled ? 'You must be a project owner' : undefined"
      :icon="PencilIcon"
    >
      <FormTextInput
        v-model="localProjectName"
        name="projectName"
        label="Project Name"
        placeholder="Project Name"
        show-label
        color="foundation"
        class="mb-4"
        :disabled="disabled"
      />
      <FormTextArea
        v-model="localProjectDescription"
        name="projectDescription"
        label="Project Description"
        placeholder="Description (optional)"
        show-label
        color="foundation"
        :disabled="disabled"
      />
      <template #bottom-buttons>
        <FormButton text :disabled="!hasChanges" @click="resetLocalState">
          Cancel
        </FormButton>
        <FormButton :disabled="!hasChanges" @click="emitUpdate">Update</FormButton>
      </template>
    </ProjectPageSettingsBlock>

    <LayoutDialog
      v-model:open="showConfirmDialog"
      max-width="md"
      :buttons="dialogButtons"
    >
      <template #header>Unsaved Changes</template>
      <div class="space-y-4">
        <p>You have unsaved changes. Do you want to save them before leaving?</p>
      </div>
    </LayoutDialog>
  </div>
</template>

<script setup lang="ts">
import { onBeforeRouteLeave, useRouter } from 'vue-router'
import { PencilIcon } from '@heroicons/vue/24/outline'
import {
  FormTextInput,
  FormTextArea,
  FormButton,
  LayoutDialog
} from '@speckle/ui-components'
import type { ProjectPageSettingsGeneralBlockProjectInfo_ProjectFragment } from '~~/lib/common/generated/gql/graphql'
import type { RouteLocationRaw } from 'vue-router'
import { graphql } from '~~/lib/common/generated/gql'

graphql(`
  fragment ProjectPageSettingsGeneralBlockProjectInfo_Project on Project {
    id
    role
    name
    description
  }
`)

const props = defineProps<{
  project: ProjectPageSettingsGeneralBlockProjectInfo_ProjectFragment
  disabled?: boolean
}>()

const emit = defineEmits(['update-project'])
const router = useRouter()

const targetRoute = ref<RouteLocationRaw | undefined>(undefined)
const localProjectName = ref(props.project.name)
const localProjectDescription = ref(props.project.description ?? '')
const showConfirmDialog = ref(false)

const hasChanges = computed(() => {
  return (
    localProjectName.value !== props.project.name ||
    localProjectDescription.value !== props.project.description
  )
})

const emitUpdate = () => {
  emit('update-project', {
    name: localProjectName.value,
    description: localProjectDescription.value,
    onComplete: handleRedirection
  })
}

const handleRedirection = () => {
  showConfirmDialog.value = false
  resetLocalState()
  if (targetRoute.value) {
    router.push(targetRoute.value)
    targetRoute.value = undefined
  }
}

const resetLocalState = () => {
  localProjectName.value = props.project.name
  localProjectDescription.value = props.project.description ?? ''
  showConfirmDialog.value = false
}

const dialogButtons = computed(() => [
  {
    text: 'Discard Changes',
    props: { color: 'secondary', fullWidth: true, outline: true },
    onClick: handleRedirection
  },
  {
    text: 'Save Changes',
    props: {
      fullWidth: true,
      outline: true,
      submit: true
    },
    onClick: () => {
      showConfirmDialog.value = false
      emitUpdate()
    }
  }
])

watch(
  () => props.project,
  (newProject, oldProject) => {
    if (newProject.name !== oldProject.name) {
      localProjectName.value = newProject.name
    }
    if (newProject.description !== oldProject.description) {
      localProjectDescription.value = newProject.description ?? ''
    }
  },
  { deep: true }
)

onBeforeRouteLeave((to, from, next) => {
  if (hasChanges.value && !showConfirmDialog.value) {
    targetRoute.value = to
    showConfirmDialog.value = true
    next(false)
  } else {
    next()
  }
})
</script>
