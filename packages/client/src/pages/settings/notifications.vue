<template>
<div class="_formRoot">
	<FormLink class="_formBlock" @click="configure"><template #icon><i class="fas fa-cog"></i></template>{{ i18n.ts.notificationSetting }}</FormLink>
	<FormSection>
		<FormLink class="_formBlock" @click="readAllNotifications">{{ i18n.ts.markAsReadAllNotifications }}</FormLink>
		<FormLink class="_formBlock" @click="readAllUnreadNotes">{{ i18n.ts.markAsReadAllUnreadNotes }}</FormLink>
		<FormLink class="_formBlock" @click="readAllMessagingMessages">{{ i18n.ts.markAsReadAllTalkMessages }}</FormLink>
	</FormSection>
</div>
</template>

<script lang="ts">
import { defineAsyncComponent, defineExpose } from 'vue';
import FormButton from '@/components/ui/button.vue';
import FormLink from '@/components/form/link.vue';
import FormSection from '@/components/form/section.vue';
import { notificationTypes } from 'misskey-js';
import * as os from '@/os';
import * as symbols from '@/symbols';
import { $i } from '@/account';
import { i18n } from '@/i18n';

async function readAllUnreadNotes() {
	await os.api('i/read-all-unread-notes');
}

async function readAllMessagingMessages() {
	await os.api('i/read-all-messaging-messages');
}

async function readAllNotifications() {
	await os.api('notifications/mark-all-as-read');
}

function configure() {
	const includingTypes = notificationTypes.filter(x => !$i!.mutingNotificationTypes.includes(x));
	os.popup(defineAsyncComponent(() => import('@/components/notification-setting-window.vue')), {
		includingTypes,
		showGlobalToggle: false,
	}, {
		done: async (res) => {
			const { includingTypes: value } = res;
			await os.apiWithDialog('i/update', {
				mutingNotificationTypes: notificationTypes.filter(x => !value.includes(x)),
			}).then(i => {
				$i!.mutingNotificationTypes = i.mutingNotificationTypes;
			});
		}
	}, 'closed');
}

defineExpose({
	[symbols.PAGE_INFO]: {
		title: i18n.ts.notifications,
		icon: 'fas fa-bell',
		bg: 'var(--bg)',
	}
});
</script>
