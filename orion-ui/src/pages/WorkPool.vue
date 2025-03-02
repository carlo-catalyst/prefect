<template>
  <p-layout-well v-if="workPool" class="work-pool">
    <template #header>
      <PageHeadingWorkPool :work-pool="workPool" @update="workPoolSubscription.refresh" />
    </template>

    <p-tabs :tabs="tabs">
      <template #details>
        <WorkPoolDetails :work-pool="workPool" />
      </template>

      <template #runs>
        <FlowRunFilteredList :flow-run-filter="flowRunFilter" />
      </template>

      <template #queues>
        <WorkPoolQueuesTable :work-pool-name="workPoolName" />
      </template>

      <template #workers>
        <WorkersTable :work-pool-filter="workPool.name" />
      </template>
    </p-tabs>

    <template #well>
      <WorkPoolDetails alternate :work-pool="workPool" />
    </template>
  </p-layout-well>
</template>

<script lang="ts" setup>
  import { useWorkspaceApi, PageHeadingWorkPool, WorkPoolDetails, WorkersTable, useRecentFlowRunFilter, FlowRunFilteredList, WorkPoolQueuesTable } from '@prefecthq/orion-design'
  import { media } from '@prefecthq/prefect-design'
  import { useRouteParam, useSubscription } from '@prefecthq/vue-compositions'
  import { computed } from 'vue'
  import { usePageTitle } from '@/compositions/usePageTitle'

  const api = useWorkspaceApi()
  const workPoolName = useRouteParam('workPoolName')

  const tabs = computed(() => {
    const values = ['Runs', 'Queues', 'Workers']

    if (!media.xl) {
      values.unshift('Details')
    }

    return values
  })

  const subscriptionOptions = {
    interval: 300000,
  }
  const workPoolSubscription = useSubscription(api.workPools.getWorkPoolByName, [workPoolName.value], subscriptionOptions)
  const workPool = computed(() => workPoolSubscription.response)
  const workPoolId = computed(() => workPool.value ? [workPool.value.id] : [])

  const flowRunFilter = useRecentFlowRunFilter({ workPools: workPoolId })

  const title = computed(() => {
    if (!workPool.value) {
      return 'Work Pool'
    }
    return `Work Pool: ${workPool.value.name}`
  })

  usePageTitle(title)
</script>