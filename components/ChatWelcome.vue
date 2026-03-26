<template>
    <div class="welcome-container">
        <div class="welcome-content">
            <div class="welcome-icon-row">
                <v-icon size="48" color="primary">mdi-atom-variant</v-icon>
            </div>
            <h1 class="welcome-title">Elementary</h1>
            <p class="welcome-subtitle">
                Your AI assistant for exploring the Elemental Knowledge Graph. Ask about entities,
                tools, schemas, or let me run queries for you.
            </p>

            <div class="suggestions-grid">
                <button
                    v-for="suggestion in suggestions"
                    :key="suggestion.text"
                    class="suggestion-chip"
                    @click="$emit('select', suggestion.text)"
                >
                    <v-icon size="18" class="suggestion-icon" :color="suggestion.color">
                        {{ suggestion.icon }}
                    </v-icon>
                    <span>{{ suggestion.text }}</span>
                </button>
            </div>
        </div>
    </div>
</template>

<script setup lang="ts">
    defineEmits<{ select: [text: string] }>();

    const suggestions = [
        {
            icon: 'mdi-toolbox-outline',
            text: 'What tools do you have?',
            color: 'primary',
        },
        {
            icon: 'mdi-magnify',
            text: 'Search for Microsoft in the knowledge graph',
            color: '#003bff',
        },
        {
            icon: 'mdi-file-tree-outline',
            text: 'Show me the schema — what entity types exist?',
            color: '#ff5c00',
        },
        {
            icon: 'mdi-lightbulb-outline',
            text: 'How would I find all companies in the tech sector?',
            color: '#ff9f0a',
        },
    ];
</script>

<style scoped>
    .welcome-container {
        display: flex;
        align-items: center;
        justify-content: center;
        height: 100%;
        padding: 24px;
    }

    .welcome-content {
        max-width: 560px;
        width: 100%;
        text-align: center;
    }

    .welcome-icon-row {
        margin-bottom: 16px;
    }

    .welcome-title {
        font-family: var(--font-headline);
        font-weight: 400;
        font-size: 2.2rem;
        letter-spacing: 0.03em;
        margin-bottom: 12px;
    }

    .welcome-subtitle {
        color: var(--lv-silver);
        font-size: 1rem;
        line-height: 1.5;
        margin-bottom: 36px;
    }

    .suggestions-grid {
        display: grid;
        grid-template-columns: 1fr 1fr;
        gap: 10px;
    }

    .suggestion-chip {
        display: flex;
        align-items: center;
        gap: 10px;
        padding: 14px 16px;
        border-radius: 12px;
        background: rgba(255, 255, 255, 0.04);
        border: 1px solid rgba(255, 255, 255, 0.08);
        color: rgba(255, 255, 255, 0.85);
        font-size: 0.85rem;
        line-height: 1.35;
        text-align: left;
        cursor: pointer;
        transition:
            background 0.15s,
            border-color 0.15s;
    }

    .suggestion-chip:hover {
        background: rgba(255, 255, 255, 0.08);
        border-color: rgba(255, 255, 255, 0.15);
    }

    .suggestion-icon {
        flex-shrink: 0;
    }

    @media (max-width: 600px) {
        .suggestions-grid {
            grid-template-columns: 1fr;
        }
    }
</style>
