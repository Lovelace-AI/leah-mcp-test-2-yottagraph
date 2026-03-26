<template>
    <div class="chat-page d-flex flex-column fill-height">
        <!-- Messages area -->
        <div ref="scrollArea" class="flex-grow-1 overflow-y-auto">
            <ChatWelcome v-if="!hasMessages" @select="handleSuggestion" />
            <div v-else class="messages-container pa-4">
                <ChatMessage v-for="msg in messages" :key="msg.id" :message="msg" />
            </div>
        </div>

        <!-- Agent status bar -->
        <div v-if="agentStatus" class="agent-status-bar flex-shrink-0 px-4 py-1">
            <v-icon size="12" :color="agentStatus.color" class="mr-1">
                {{ agentStatus.icon }}
            </v-icon>
            <span class="text-caption" :style="{ color: agentStatus.textColor }">
                {{ agentStatus.text }}
            </span>
        </div>

        <!-- Input area -->
        <div class="input-area flex-shrink-0 pa-4">
            <div class="input-wrapper">
                <v-textarea
                    v-model="inputText"
                    placeholder="Ask Elementary anything..."
                    variant="outlined"
                    density="comfortable"
                    rows="1"
                    max-rows="5"
                    auto-grow
                    hide-details
                    color="primary"
                    :disabled="loading"
                    @keydown.enter.exact.prevent="handleSend"
                >
                    <template #append-inner>
                        <v-btn
                            icon
                            size="small"
                            variant="text"
                            color="primary"
                            :disabled="!inputText.trim() || loading"
                            @click="handleSend"
                        >
                            <v-icon>mdi-send</v-icon>
                        </v-btn>
                    </template>
                </v-textarea>
            </div>
            <div class="text-caption text-medium-emphasis text-center mt-2" style="opacity: 0.5">
                Elementary uses the Elemental Knowledge Graph to answer your queries.
            </div>
        </div>
    </div>
</template>

<script setup lang="ts">
    import { useTenantConfig, type AgentConfig } from '~/composables/useTenantConfig';
    import { useAgentChat } from '~/composables/useAgentChat';

    const scrollArea = ref<HTMLElement | null>(null);
    const inputText = ref('');
    const agentResolved = ref(false);

    const { config, fetchConfig } = useTenantConfig();
    const { messages, loading, hasMessages, sendMessage, selectAgent, currentAgentId } =
        useAgentChat();

    const agentStatus = computed(() => {
        if (!agentResolved.value) {
            return {
                icon: 'mdi-loading mdi-spin',
                text: 'Connecting to agent...',
                color: 'warning',
                textColor: 'var(--lv-silver)',
            };
        }
        if (!currentAgentId.value) {
            return {
                icon: 'mdi-alert-circle-outline',
                text: 'No agent deployed yet — deploy with /deploy_agent in Cursor',
                color: 'warning',
                textColor: 'var(--lv-orange)',
            };
        }
        return null;
    });

    async function resolveAgent() {
        try {
            const tenantCfg = await fetchConfig();
            const agents: AgentConfig[] = tenantCfg?.agents ?? [];
            if (agents.length > 0) {
                selectAgent(agents[0].engine_id);
            }
        } catch {
            // Config fetch failed — agent status will show the warning
        } finally {
            agentResolved.value = true;
        }
    }

    function handleSend() {
        const text = inputText.value.trim();
        if (!text || loading.value) return;
        inputText.value = '';
        sendMessage(text);
    }

    function handleSuggestion(text: string) {
        inputText.value = text;
        handleSend();
    }

    watch(
        messages,
        async () => {
            await nextTick();
            if (scrollArea.value) {
                scrollArea.value.scrollTop = scrollArea.value.scrollHeight;
            }
        },
        { deep: true }
    );

    onMounted(() => {
        resolveAgent();
    });
</script>

<style scoped>
    .chat-page {
        height: 100%;
    }

    .messages-container {
        max-width: 800px;
        margin: 0 auto;
        padding-bottom: 8px;
    }

    .agent-status-bar {
        display: flex;
        align-items: center;
        justify-content: center;
        background: rgba(255, 255, 255, 0.02);
        border-top: 1px solid rgba(255, 255, 255, 0.05);
    }

    .input-area {
        border-top: 1px solid rgba(255, 255, 255, 0.06);
        background: var(--lv-black);
    }

    .input-wrapper {
        max-width: 800px;
        margin: 0 auto;
    }
</style>
