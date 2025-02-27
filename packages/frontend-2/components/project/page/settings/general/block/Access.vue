<template>
  <ProjectPageSettingsBlock
    background
    title="Access"
    :icon="LockClosedIcon"
    :disabled-message="disabled ? 'You must be a project owner' : undefined"
  >
    <template #introduction>
      <p>Choose how you want to share this project with others.</p>
    </template>
    <FormRadioGroup
      v-model="selectedOption"
      :options="radioOptions"
      :disabled="disabled"
      @update:model-value="emitUpdate"
    />
  </ProjectPageSettingsBlock>
</template>

<script setup lang="ts">
import { LockClosedIcon, LinkIcon, GlobeAltIcon } from '@heroicons/vue/24/outline'
import { FormRadioGroup } from '@speckle/ui-components'
import { ProjectVisibility } from '~/lib/common/generated/gql/graphql'
import { graphql } from '~~/lib/common/generated/gql'
import type { ProjectPageSettingsGeneralBlockAccess_ProjectFragment } from '~~/lib/common/generated/gql/graphql'

graphql(`
  fragment ProjectPageSettingsGeneralBlockAccess_Project on Project {
    id
    visibility
  }
`)

const props = defineProps<{
  project: ProjectPageSettingsGeneralBlockAccess_ProjectFragment
  disabled?: boolean
}>()

const emit = defineEmits<{
  (e: 'update-visibility', v: ProjectVisibility): void
}>()

const selectedOption = ref(props.project.visibility || ProjectVisibility.Private)

const radioOptions = computed(() => [
  {
    value: ProjectVisibility.Public,
    title: 'Public',
    introduction: 'Anyone can view and access your project',
    icon: GlobeAltIcon
  },
  {
    value: ProjectVisibility.Unlisted,
    title: 'Unlisted',
    introduction: 'Anyone with the link can access',
    icon: LinkIcon
  },
  {
    value: ProjectVisibility.Private,
    title: 'Private',
    introduction: 'Only collaborators can access',
    icon: LockClosedIcon
  }
])

watch(
  () => props.project.visibility,
  (newVal) => {
    selectedOption.value = newVal ?? ProjectVisibility.Private
  }
)

const emitUpdate = (value: ProjectVisibility) => {
  emit('update-visibility', value)
}
</script>
