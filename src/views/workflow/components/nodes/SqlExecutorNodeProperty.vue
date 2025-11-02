<script setup lang="ts">
import {NInput} from 'naive-ui'
import NodePropertyInput from '../NodePropertyInput.vue'
import {onMounted} from "vue";

const props = defineProps<Props>()


interface Props {
	workflow: Workflow.WorkflowInfo
	wfNode: Workflow.WorkflowNode
}

const nodeConfig = props.wfNode.nodeConfig as Workflow.NodeConfigSql




onMounted(() => {
	console.log('nodeConfig', nodeConfig.sqlListStr)

})
</script>

<template>
	<div class="flex flex-col w-full">
		<NodePropertyInput :workflow="workflow" :wf-node="wfNode"/>
		<div class="mt-6">
			<div class="text-xl mb-1 flex align-center">
				URL
			</div>
			<div>
				<NInput v-model:value="nodeConfig.sqlListStr"/>
			</div>
		</div>
	</div>
</template>
