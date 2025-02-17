<script setup>
import PageBar from '@/views/components/PageBar.vue'
import FilledButton from '@/views/components/FilledButton.vue'
import { useRouter } from 'vue-router'
import TableHeader from '@/views/components/Table/TableHeader.vue'
import TableMessage from '@/views/components/Table/TableMessage.vue'
import Table from '@/views/components/Table/Table.vue'
import { useQuery } from '@vue/apollo-composable'
import gql from 'graphql-tag'
import { computed } from 'vue'
import { useToast } from 'vue-toastification'
import ApplicationGroup from '@/views/partials/ApplicationGroup.vue'

const router = useRouter()
const toast = useToast()

const deployNewApplication = () => {
  router.push('/deploy/application')
}

const {
  result: applicationsResult,
  refetch: refetchApplications,
  loading: isApplicationsLoading,
  onError: onApplicationsError
} = useQuery(
  gql`
    query {
      applications {
        id
        name
        deploymentMode
        replicas
        isSleeping
        realtimeInfo {
          InfoFound
          DesiredReplicas
          RunningReplicas
          DeploymentMode
        }
        latestDeployment {
          status
          upstreamType
          gitProvider
          createdAt
        }
        group
      }
    }
  `,
  null,
  {
    pollInterval: 30000
  }
)

onApplicationsError((err) => {
  toast.error(err.message)
})

const applications = computed(() => applicationsResult.value?.applications ?? [])
const applicationGroupWise = computed(() => {
  let applications = applicationsResult.value?.applications ?? []
  let groupedApplications = {}
  applications.forEach((application) => {
    if (!groupedApplications[encodeURI(application.group)]) {
      groupedApplications[encodeURI(application.group)] = []
    }
    groupedApplications[encodeURI(application.group)].push(application)
  })
  return groupedApplications
})
</script>

<template>
  <section class="mx-auto w-full max-w-7xl">
    <!-- Top Page bar   -->
    <PageBar>
      <template v-slot:title>Deployed Applications</template>
      <template v-slot:subtitle>Take control of your deployed applications</template>
      <template v-slot:buttons>
        <FilledButton :click="deployNewApplication" type="primary">
          <font-awesome-icon icon="fa-solid fa-hammer" class="mr-2" />
          Deploy App
        </FilledButton>
        <FilledButton type="ghost" :click="refetchApplications">
          <font-awesome-icon
            icon="fa-solid fa-arrows-rotate"
            :class="{
              'animate-spin ': isApplicationsLoading
            }" />&nbsp;&nbsp; Refresh List
        </FilledButton>
      </template>
    </PageBar>

    <!-- Table -->
    <Table class="mt-8">
      <template v-slot:header>
        <TableHeader align="left">Application / Group</TableHeader>
        <TableHeader align="center">Status</TableHeader>
        <TableHeader align="center">Replicas</TableHeader>
        <TableHeader align="center">Source</TableHeader>
        <TableHeader align="center">Last Deployment</TableHeader>
        <TableHeader align="right">Details</TableHeader>
      </template>
      <template v-slot:message>
        <TableMessage v-if="applications.length === 0">
          No deployed applications found.<br />
          Click on the "Deploy New" button to deploy a new application.
        </TableMessage>
        <TableMessage v-if="isApplicationsLoading"> Loading deployed applications...</TableMessage>
      </template>
      <template v-slot:body>
        <ApplicationGroup
          v-for="(applications, group, index) in applicationGroupWise"
          :key="group"
          :group="group"
          :group-index="index"
          :applications="applications" />
      </template>
    </Table>
  </section>
</template>

<style scoped></style>
