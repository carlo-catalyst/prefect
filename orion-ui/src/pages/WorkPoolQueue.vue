<template>
  <p-layout-default v-if="workPoolQueue" class="work-pool-queue">
    <template #header>
      <PageHeadingWorkPoolQueue :work-pool-queue="workPoolQueue" :work-pool-name="workPoolName" @update="workPoolQueuesSubscription.refresh" />
    </template>

    <p-layout-well class="work-pool-queue__body">
      <template #header>
        <!-- Update banner with correct information -->
        <CodeBanner :command="workPoolQueueCliCommand" title="Work pool queue is ready to go!" subtitle="Work Pool Queue description" />
      </template>

      <p-tabs :tabs="tabs">
        <template #details>
          <WorkPoolQueueDetails :work-pool-name="workPoolName" :work-pool-queue="workPoolQueue" />
        </template>

        <template #upcoming-runs>
          <WorkPoolQueueUpcomingFlowRunsList :work-pool-name="workPoolName" :work-pool-queue="workPoolQueue" />
        </template>

        <template #runs>
          <FlowRunFilteredList :flow-run-filter="flowRunFilter" />
        </template>
      </p-tabs>

      <template #well>
        <WorkPoolQueueDetails alternate :work-pool-name="workPoolName" :work-pool-queue="workPoolQueue" />
      </template>
    </p-layout-well>
  </p-layout-default>
</template>

<script lang="ts" setup>
  import { useWorkspaceApi, PageHeadingWorkPoolQueue, CodeBanner, WorkPoolQueueDetails, WorkPoolQueueUpcomingFlowRunsList, useFlowRunFilter, FlowRunFilteredList } from '@prefecthq/orion-design'
  import { media } from '@prefecthq/prefect-design'
  import { useRouteParam, useSubscription } from '@prefecthq/vue-compositions'
  import { computed } from 'vue'
  import { usePageTitle } from '@/compositions/usePageTitle'

  const api = useWorkspaceApi()
  const workPoolName = useRouteParam('workPoolName')
  const workPoolQueueName = useRouteParam('workPoolQueueName')
  const subscriptionOptions = {
    interval: 300000,
  }

  const workPoolQueuesSubscription = useSubscription(api.workPoolQueues.getWorkPoolQueueByName, [workPoolName.value, workPoolQueueName.value], subscriptionOptions)
  const workPoolQueue = computed(() => workPoolQueuesSubscription.response)

  const workPoolQueueCliCommand = computed(() => 'code snippet for work pool queue')

  const flowRunFilter = useFlowRunFilter({ workPoolQueueName: [workPoolQueueName.value] })

  const tabs = computed(() => {
    const values = ['Upcoming Runs', 'Runs']

    if (!media.xl) {
      values.unshift('Details')
    }

    return values
  })

  const title = computed(() => {
    if (!workPoolQueueName.value) {
      return 'Work Pool Queue'
    }
    return `Work Pool Queue: ${workPoolQueueName.value}`
  })

  usePageTitle(title)
</script>

<style>
/* This is an override since this is using nested layouts */
.work-pool-queue__body {
  @apply
  p-0
}
</style>