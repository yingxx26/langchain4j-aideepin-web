<script setup lang="ts">
import {NInput} from 'naive-ui'
import NodePropertyInput from '../NodePropertyInput.vue'
import {onMounted} from "vue";

const props = defineProps<Props>()


interface Props {
	workflow: Workflow.WorkflowInfo
	wfNode: Workflow.WorkflowNode
}

const nodeConfig = props.wfNode.nodeConfig as Workflow.NodeConfigDbTable


onMounted(() => {

})
</script>

<template>
	<div class="flex flex-col w-full">
		<NodePropertyInput :workflow="workflow" :wf-node="wfNode"/>
		<div class="mt-6">
			<div class="text-xl mb-1 flex align-center">
				dburl
			</div>
			<div>
				<NInput v-model:value="nodeConfig.dburl"/>
			</div>
		</div>
		<div class="mt-6">
			<div class="text-xl mb-1 flex align-center">
				dbuser
			</div>
			<div>
				<NInput v-model:value="nodeConfig.dbuser"/>
			</div>
		</div>
		<div class="mt-6">
			<div class="text-xl mb-1 flex align-center">
				dbpassword
			</div>
			<div>
				<NInput v-model:value="nodeConfig.dbpassword"/>
			</div>
		</div>
		<div class="mt-6">
			<div class="text-xl mb-1 flex align-center">
				dbtable
			</div>
			<div>
				<NInput v-model:value="nodeConfig.dbtable"/>
			</div>
		</div>
	</div>
</template>
