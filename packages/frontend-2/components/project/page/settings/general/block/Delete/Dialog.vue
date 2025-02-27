<template>
  <LayoutDialog v-model:open="isOpen" max-width="md" :buttons="dialogButtons">
    <template #header>Delete Project</template>
    <div class="space-y-4">
      <p>
        Are you sure you want to permanently
        <strong>delete “{{ project.name }}”</strong>
        and all its contents, including
        <strong>({{ project.models.totalCount }}) {{ modelText }}</strong>
        <span v-if="project.commentThreads.totalCount">
          and
          <strong>
            ({{ project.commentThreads.totalCount }}) {{ discussionText }}
          </strong>
        </span>
        ?
      </p>
      <p>To confirm deletion, type the project name below and press Delete.</p>
      <FormTextInput
        v-model="projectNameInput"
        name="projectNameConfirm"
        label="Project name"
        placeholder="Type the project name here"
        full-width
        hide-error-message
        class="text-sm"
        color="foundation"
      />
    </div>
  </LayoutDialog>
</template>

<script setup lang="ts">
import { LayoutDialog, FormTextInput } from '@speckle/ui-components'
import { useDeleteProject } from '~~/lib/projects/composables/projectManagement'
import { useMixpanel } from '~~/lib/core/composables/mp'
import type { ProjectPageSettingsGeneralBlockDelete_ProjectFragment } from '~~/lib/common/generated/gql/graphql'
import { Roles } from '@speckle/shared'

const isOpen = defineModel<boolean>('open', { required: true })

const props = defineProps<{
  project: ProjectPageSettingsGeneralBlockDelete_ProjectFragment
}>()

const projectNameInput = ref('')

const deleteProject = useDeleteProject()
const mp = useMixpanel()

const modelText = computed(() =>
  props.project.models.totalCount === 1 ? 'model' : 'models'
)
const discussionText = computed(() =>
  props.project.commentThreads.totalCount === 1 ? 'discussion' : 'discussions'
)

const dialogButtons = computed(() => [
  {
    text: 'Cancel',
    props: { color: 'secondary', fullWidth: true },
    onClick: () => {
      isOpen.value = false
      projectNameInput.value = ''
    }
  },
  {
    text: 'Delete',
    props: {
      color: 'danger',
      fullWidth: true,
      outline: true,
      disabled: projectNameInput.value !== props.project.name
    },
    onClick: async () => {
      if (
        projectNameInput.value === props.project.name &&
        props.project.role === Roles.Stream.Owner
      ) {
        await deleteProject(props.project.id, { goHome: true })
        isOpen.value = false
        mp.track('Stream Action', { type: 'action', name: 'delete' })
      }
    }
  }
])
</script>
