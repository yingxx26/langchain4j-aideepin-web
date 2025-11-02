<script setup lang="ts">

import {Handle, Position} from '@vue-flow/core'
import type {NodeProps} from '@vue-flow/core'
import CommonNodeHeader from '../CommonNodeHeader.vue'

defineProps<NodeProps>()
</script>

<template>
	<div class="flex flex-col w-full">
		<Handle type="source" :position="Position.Right"/>
		<Handle type="target" :position="Position.Left"/>
		<CommonNodeHeader :wf-node="data"/>
		<div clas="flex-1 flex-col">
			<div class="content_line flex items-center">
				dburl： {{ data.nodeConfig.dburl }}
			</div>
			<div class="content_line flex items-center">
				dbuser： {{ data.nodeConfig.dbuser }}
			</div>
			<div class="content_line flex items-center">
				dbpassword： {{ data.nodeConfig.dbpassword }}
			</div>
			<div class="content_line flex items-center">
				dbtable： {{ data.nodeConfig.dbtable }}
			</div>
		</div>
	</div>
</template>
