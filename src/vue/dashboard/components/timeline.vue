<script lang="ts">
import type { AttachmentHistory } from 'zotero-reading-history';
import { toTimeString } from '@/utility/utils';
import HistoryAnalyzer from '@/utility/history';

export default {
    props: {
        history: {
            type: Array<AttachmentHistory>,
            required: true,
        },
    },
    watch: {
        history(newHis: AttachmentHistory[]) {
            this.items.length = 0;
            const ha = new HistoryAnalyzer(newHis),
                attachments = ha.validAttachments;
            attachments.forEach(att => {
                const title =
                    attachments.length > 1
                        ? (att.getField('title') as string)
                        : undefined;
                this.items.push(
                    {
                        dot: 'default',
                        date: new Date(att.dateAdded),
                        content: toolkit.locale.dateAdded,
                        title,
                    },
                    {
                        dot: 'warning',
                        date: new Date(att.dateModified),
                        content: toolkit.locale.dateModified,
                        title,
                    }
                );
            });
            newHis.forEach(his => {
                const ha = new HistoryAnalyzer([his]),
                    dateTimeStats = ha.dateTimeStats;
                dateTimeStats.forEach(({ date, time }) => {
                    this.items.push({
                        dot: 'primary',
                        date: new Date(date),
                        content: toTimeString(time),
                        title:
                            attachments.length > 1 ? ha.titles[0] : undefined,
                    });
                });
            });
            this.items = this.items.sort(
                (a, b) => a.date.getTime() - b.date.getTime()
            );
        },
    },
    data() {
        return {
            items: new Array<{
                dot: 'default' | 'primary' | 'warning';
                date: Date;
                content: string;
                title?: string;
            }>(),
        };
    },
};
</script>

<template>
    <div class="timeline">
        <t-timeline style="margin: auto">
            <t-timeline-item
                v-for="item of items"
                :label="item.date.toLocaleDateString()"
                :dot-color="item.dot"
            >
                <t-tag
                    v-if="item.title"
                    theme="success"
                    variant="light"
                    max-width="220px"
                    >{{ item.title }}</t-tag
                >
                <p>{{ item.content }}</p>
            </t-timeline-item>
        </t-timeline>
    </div>
</template>

<style scoped>
.timeline {
    display: flex;
    align-content: center;
    padding: 10px;
}
</style>
