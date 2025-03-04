<script setup lang="ts">
import { ScalarIcon } from '@scalar/components'
import type { Target, TargetId } from '@scalar/snippetz/types'
import { ref } from 'vue'

import { type HttpClientState, useHttpClientStore } from '../../../stores'

// Use the template store to keep it accessible globally
const {
  httpClient,
  setHttpClient,
  availableTargets,
  getClientTitle,
  getTargetTitle,
} = useHttpClientStore()

const containerRef = ref<HTMLElement>()

// Show popular clients with an icon, not just in a select.
const featuredClients = (
  [
    {
      targetKey: 'shell',
      clientKey: 'curl',
    },
    {
      targetKey: 'ruby',
      clientKey: 'native',
    },
    {
      targetKey: 'node',
      clientKey: 'undici',
    },
    {
      targetKey: 'php',
      clientKey: 'guzzle',
    },
    {
      targetKey: 'python',
      clientKey: 'python3',
    },
  ] as const
).filter((featuredClient) =>
  availableTargets.value.find((target: Target) => {
    return (
      target.key === featuredClient.targetKey &&
      target.clients.find(
        (client) => client.client === featuredClient.clientKey,
      )
    )
  }),
)

/**
 * Icons have longer names to appear in icon searches, e.g. "javascript-js" instead of just "javascript". This function
 * maps the language key to the icon name.
 */
const getIconByLanguageKey = (targetKey: TargetId) =>
  `programming-language-${targetKey === 'js' ? 'javascript' : targetKey}` as const

const isSelectedClient = (language: HttpClientState) => {
  return (
    language.targetKey === httpClient.targetKey &&
    language.clientKey === httpClient.clientKey
  )
}

const checkIfClientIsFeatured = (client: HttpClientState) =>
  featuredClients.some(
    (item) =>
      item.targetKey === client.targetKey &&
      item.clientKey === client.clientKey,
  )
</script>
<template>
  <div
    ref="containerRef"
    class="client-libraries-content">
    <button
      v-for="client in featuredClients"
      :key="client.clientKey"
      aria-hidden="true"
      class="client-libraries rendered-code-sdks"
      :class="{
        'client-libraries__active': isSelectedClient(client),
      }"
      tabindex="-1"
      type="button"
      @click="() => setHttpClient(client)">
      <div :class="`client-libraries-icon__${client.targetKey}`">
        <ScalarIcon
          class="client-libraries-icon"
          :icon="getIconByLanguageKey(client.targetKey)" />
      </div>
      <span>{{ getTargetTitle(client) }}</span>
    </button>

    <label
      class="client-libraries client-libraries__select"
      :class="{
        'client-libraries__active':
          httpClient && !checkIfClientIsFeatured(httpClient),
      }">
      <span class="sr-only">Select Client Library</span>
      <select
        class="language-select"
        :value="JSON.stringify(httpClient)"
        @input="
          (event) =>
            setHttpClient(JSON.parse((event.target as HTMLSelectElement).value))
        ">
        <optgroup
          v-for="target in availableTargets"
          :key="target.key"
          :label="target.title">
          <option
            v-for="client in target.clients"
            :key="client.client"
            :aria-label="`${target.title} ${getClientTitle({
              targetKey: target.key,
              clientKey: client.client,
            })}`"
            :value="
              JSON.stringify({
                targetKey: target.key,
                clientKey: client.client,
              })
            ">
            {{
              getClientTitle({
                targetKey: target.key,
                clientKey: client.client,
              })
            }}
          </option>
        </optgroup>
      </select>
      <div
        aria-hidden="true"
        class="client-libraries-icon__more">
        <template v-if="httpClient && !checkIfClientIsFeatured(httpClient)">
          <div :class="`client-libraries-icon__${httpClient.targetKey}`">
            <ScalarIcon
              class="client-libraries-icon"
              :icon="getIconByLanguageKey(httpClient.targetKey)" />
          </div>
        </template>
        <template v-else>
          <svg
            class="client-libraries-icon"
            height="50"
            role="presentation"
            viewBox="0 0 50 50"
            width="50"
            xmlns="http://www.w3.org/2000/svg">
            <g
              fill="currentColor"
              fill-rule="nonzero">
              <path
                d="M10.71 25.3a3.87 3.87 0 1 0 7.74 0 3.87 3.87 0 0 0-7.74 0M21.13 25.3a3.87 3.87 0 1 0 7.74 0 3.87 3.87 0 0 0-7.74 0M31.55 25.3a3.87 3.87 0 1 0 7.74 0 3.87 3.87 0 0 0-7.74 0" />
            </g>
          </svg>
        </template>
      </div>
      <span
        v-if="availableTargets.length"
        aria-hidden="true">
        More
      </span>
    </label>
  </div>
</template>
<style scoped>
.client-libraries-content {
  container: client-libraries-content / inline-size;
  display: flex;
  justify-content: center;
  overflow: hidden;
  background-color: var(--scalar-background-2);
  border-radius: var(--scalar-radius) var(--scalar-radius) 0 0;
  border: var(--scalar-border-width) solid var(--scalar-border-color);
}
.client-libraries {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 100%;
  position: relative;
  cursor: pointer;
  white-space: nowrap;
  padding: 8px 2px;
  gap: 6px;
  color: var(--scalar-color-3);
  user-select: none;
}
.client-libraries:first-child {
  border-radius: var(--scalar-radius) 0 0 0;
}

.client-libraries:not(.client-libraries__active):hover:before {
  content: '';
  position: absolute;
  width: calc(100% - 4px);
  height: calc(100% - 4px);
  background: var(--scalar-background-3);
  left: 2px;
  top: 2px;
  z-index: 0;
  border-radius: var(--scalar-radius);
}
.client-libraries:active {
  color: var(--scalar-color-1);
}
.client-libraries:focus-visible {
  outline: none;
  box-shadow: inset 0 0 0 1px var(--scalar-color-accent);
}
/* remove php and c on mobile */
@media screen and (max-width: 450px) {
  .client-libraries:nth-of-type(4),
  .client-libraries:nth-of-type(5) {
    display: none;
  }
}
.client-libraries-icon {
  max-width: 14px;
  max-height: 14px;
  min-width: 14px;
  width: 100%;
  aspect-ratio: 1;
  display: flex;
  align-items: center;
  justify-content: center;
  position: relative;
  box-sizing: border-box;
  color: currentColor;
}
.client-libraries-icon__more svg {
  height: initial;
}
@container client-libraries-content (width < 400px) {
  .client-libraries__select {
    width: fit-content;

    .client-libraries-icon__more + span {
      display: none;
    }
  }
}
@container client-libraries-content (width < 380px) {
  .client-libraries {
    width: 100%;
  }
  .client-libraries span {
    display: none;
  }
}
.client-libraries__active {
  color: var(--scalar-color-1);
  border-bottom: var(--scalar-border-width) solid var(--scalar-color-1);
}
@keyframes codeloader {
  0% {
    transform: rotate(0deg);
  }
  to {
    transform: rotate(1turn);
  }
}
.client-libraries span {
  font-size: var(--scalar-mini);
  font-weight: var(--scalar-semibold);
  position: relative;
}
.client-libraries__active span {
  color: var(--scalar-color-1);
}
.client-libraries__select select {
  background: var(--scalar-background-3);
  color: var(--scalar-color-2);
  opacity: 0;
  height: 100%;
  width: 100%;
  aspect-ratio: 1;
  position: absolute;
  top: 0;
  left: 0;
  cursor: pointer;
  z-index: 1;
  appearance: none;
  border: none;
}
.client-libraries__select:has(select:focus-visible) {
  border-radius: var(--scalar-radius);
  box-shadow: inset 0 0 0 1px var(--scalar-color-accent);
}
.client-libraries__select span {
  position: relative;
  display: flex;
  align-items: center;
}
@media screen and (max-width: 600px) {
  .references-classic .client-libraries {
    flex-direction: column;
  }
}
</style>
