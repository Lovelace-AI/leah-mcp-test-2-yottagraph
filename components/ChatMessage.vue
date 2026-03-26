<template>
    <div class="d-flex mb-4" :class="isUser ? 'justify-end' : 'justify-start'">
        <div
            class="chat-bubble pa-3 rounded-lg"
            :class="[isUser ? 'user-bubble' : 'agent-bubble', message.error ? 'error-bubble' : '']"
            style="max-width: 80%; word-break: break-word"
        >
            <div v-if="!isUser" class="d-flex align-center mb-1">
                <v-icon size="16" class="mr-1" :color="message.error ? 'error' : 'primary'">
                    {{ message.error ? 'mdi-alert-circle' : 'mdi-atom-variant' }}
                </v-icon>
                <span class="text-caption text-medium-emphasis">Elementary</span>
            </div>

            <div v-if="message.streaming && !message.text" class="typing-indicator">
                <span /><span /><span />
            </div>
            <div v-else-if="isUser" class="text-body-2" style="white-space: pre-wrap">
                {{ message.text }}
            </div>
            <div v-else class="message-markdown text-body-2" v-html="renderedHtml" />

            <div class="text-caption text-medium-emphasis mt-1" style="opacity: 0.6">
                {{ formatTime(message.timestamp) }}
            </div>
        </div>
    </div>
</template>

<script setup lang="ts">
    import type { ChatMessage } from '~/composables/useAgentChat';

    const props = defineProps<{ message: ChatMessage }>();
    const isUser = computed(() => props.message.role === 'user');

    const renderedHtml = computed(() => {
        if (isUser.value) return '';
        return renderMarkdown(props.message.text || '');
    });

    function renderMarkdown(text: string): string {
        let html = escapeHtml(text);

        // Code blocks (```...```)
        html = html.replace(/```(\w*)\n([\s\S]*?)```/g, (_m, _lang, code) => {
            return `<pre><code>${code.trim()}</code></pre>`;
        });

        // Inline code
        html = html.replace(/`([^`]+)`/g, '<code>$1</code>');

        // Bold
        html = html.replace(/\*\*(.+?)\*\*/g, '<strong>$1</strong>');

        // Italic
        html = html.replace(/\*(.+?)\*/g, '<em>$1</em>');

        // Headings (### / ## / #)
        html = html.replace(/^### (.+)$/gm, '<h4>$1</h4>');
        html = html.replace(/^## (.+)$/gm, '<h3>$1</h3>');
        html = html.replace(/^# (.+)$/gm, '<h2>$1</h2>');

        // Unordered lists
        html = html.replace(/^[-*] (.+)$/gm, '<li>$1</li>');
        html = html.replace(/((?:<li>.*<\/li>\n?)+)/g, '<ul>$1</ul>');

        // Numbered lists
        html = html.replace(/^\d+\. (.+)$/gm, '<li>$1</li>');

        // Line breaks (preserve paragraph spacing)
        html = html.replace(/\n{2,}/g, '</p><p>');
        html = html.replace(/\n/g, '<br>');

        return `<p>${html}</p>`;
    }

    function escapeHtml(text: string): string {
        const map: Record<string, string> = {
            '&': '&amp;',
            '<': '&lt;',
            '>': '&gt;',
        };
        return text.replace(/[&<>]/g, (ch) => map[ch]);
    }

    function formatTime(ts: number): string {
        return new Date(ts).toLocaleTimeString('en-US', { hour: '2-digit', minute: '2-digit' });
    }
</script>

<style scoped>
    .user-bubble {
        background: rgba(63, 234, 0, 0.1);
        border: 1px solid rgba(63, 234, 0, 0.2);
    }
    .agent-bubble {
        background: rgba(255, 255, 255, 0.04);
        border: 1px solid rgba(255, 255, 255, 0.08);
    }
    .error-bubble {
        background: rgba(239, 68, 68, 0.1);
        border: 1px solid rgba(239, 68, 68, 0.25);
    }

    .streaming-cursor {
        display: inline-block;
        width: 2px;
        height: 1em;
        background: currentColor;
        margin-left: 1px;
        vertical-align: text-bottom;
        animation: cursor-blink 0.8s steps(2) infinite;
    }

    @keyframes cursor-blink {
        0% {
            opacity: 1;
        }
        100% {
            opacity: 0;
        }
    }

    .typing-indicator {
        display: flex;
        gap: 4px;
        padding: 4px 0;
    }

    .typing-indicator span {
        width: 6px;
        height: 6px;
        border-radius: 50%;
        background: rgba(255, 255, 255, 0.4);
        animation: typing-bounce 1.2s ease-in-out infinite;
    }

    .typing-indicator span:nth-child(2) {
        animation-delay: 0.15s;
    }

    .typing-indicator span:nth-child(3) {
        animation-delay: 0.3s;
    }

    @keyframes typing-bounce {
        0%,
        60%,
        100% {
            transform: translateY(0);
            opacity: 0.4;
        }
        30% {
            transform: translateY(-4px);
            opacity: 1;
        }
    }

    /* Markdown content styles */
    .message-markdown :deep(h2),
    .message-markdown :deep(h3),
    .message-markdown :deep(h4) {
        font-family: var(--font-headline);
        font-weight: 400;
        margin: 12px 0 6px;
    }

    .message-markdown :deep(h2) {
        font-size: 1.15rem;
    }
    .message-markdown :deep(h3) {
        font-size: 1.05rem;
    }
    .message-markdown :deep(h4) {
        font-size: 0.95rem;
    }

    .message-markdown :deep(pre) {
        background: rgba(0, 0, 0, 0.4);
        border: 1px solid rgba(255, 255, 255, 0.08);
        border-radius: 8px;
        padding: 12px 14px;
        margin: 8px 0;
        overflow-x: auto;
        font-size: 0.82rem;
        line-height: 1.5;
    }

    .message-markdown :deep(pre code) {
        background: none;
        padding: 0;
        color: inherit;
        font-size: inherit;
    }

    .message-markdown :deep(code) {
        background: rgba(63, 234, 0, 0.1);
        padding: 1px 5px;
        border-radius: 4px;
        font-size: 0.85em;
        color: var(--lv-green);
    }

    .message-markdown :deep(ul) {
        padding-left: 20px;
        margin: 6px 0;
    }

    .message-markdown :deep(li) {
        margin: 3px 0;
    }

    .message-markdown :deep(strong) {
        color: rgba(255, 255, 255, 0.95);
    }

    .message-markdown :deep(p) {
        margin: 0;
    }

    .message-markdown :deep(p + p) {
        margin-top: 8px;
    }
</style>
