<script setup lang="ts">
import { useI18n } from "vue-i18n";
import { Cloud, Copy, Globe, Server } from "lucide-vue-next";
import { Card, CardContent, CardHeader, CardTitle } from "@/components/ui/card";
import type { DDNSDualGatewayPayload } from "@/lib/api/ddns";

defineProps<{
  data: DDNSDualGatewayPayload | null;
  loading?: boolean;
}>();

const { t } = useI18n();

async function copyText(value: string | undefined | null) {
  if (!value) return;
  try {
    await navigator.clipboard.writeText(value);
  } catch {
    // ignore
  }
}
</script>

<template>
  <Card class="overflow-hidden py-5 mb-6">
    <CardHeader>
      <div class="flex items-center justify-between">
        <CardTitle class="text-base font-medium flex items-center gap-2">
          <Globe class="h-4 w-4" />
          {{ t("admin.ddns.dualGatewayTitle") }}
        </CardTitle>
        <span
          v-if="data"
          class="inline-flex items-center rounded-md px-2 py-0.5 text-xs font-medium"
          :class="
            data.ready
              ? 'bg-green-500/10 text-green-600 dark:text-green-400'
              : 'bg-amber-500/10 text-amber-600 dark:text-amber-400'
          "
        >
          {{ data.ready ? t("admin.ddns.dualGatewayReady") : data.message }}
        </span>
      </div>
      <p
        v-if="!loading && data"
        class="mt-1 text-xs text-muted-foreground"
      >
        {{ t("admin.ddns.dualGatewayDescription") }}
      </p>
    </CardHeader>

    <CardContent v-if="data">
      <div class="mb-4 flex items-center gap-2">
        <button
          type="button"
          class="inline-flex items-center gap-1.5 rounded-md bg-primary/10 px-2.5 py-1 text-sm font-mono font-medium text-primary transition-colors hover:bg-primary/15 focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-ring"
          :aria-label="
            t('common.copyAddress', { address: data.unifiedHostname })
          "
          @click="copyText(data.unifiedHostname)"
        >
          <Copy class="h-3.5 w-3.5" />
          {{ data.unifiedHostname || t("admin.ddns.notConfigured") }}
        </button>
      </div>

      <div class="grid gap-4 md:grid-cols-2">
        <!-- 网关 A: IPv6 直连 -->
        <div
          class="rounded-xl border p-4"
          :class="
            data.ipv6Direct.enabled
              ? 'border-green-500/30 bg-green-500/5'
              : 'border-muted bg-muted/30'
          "
        >
          <div class="mb-3 flex items-center gap-2">
            <div
              class="p-1.5 rounded-lg"
              :class="
                data.ipv6Direct.enabled
                  ? 'bg-green-500/15 text-green-600 dark:text-green-400'
                  : 'bg-muted text-muted-foreground'
              "
            >
              <Server class="h-4 w-4" />
            </div>
            <div class="min-w-0">
              <p class="text-sm font-medium truncate">
                {{ data.ipv6Direct.label }}
              </p>
              <p class="text-[11px] text-muted-foreground">AAAA → DDNS 直连</p>
            </div>
            <span
              class="ml-auto inline-flex h-2 w-2 rounded-full"
              :class="
                data.ipv6Direct.enabled
                  ? 'bg-green-500'
                  : 'bg-muted-foreground/30'
              "
            />
          </div>
          <p class="text-xs text-muted-foreground mb-2">
            {{ data.ipv6Direct.description }}
          </p>
          <div class="space-y-1">
            <div class="flex items-center gap-2 text-xs">
              <span class="text-muted-foreground">IPv6</span>
              <button
                v-if="data.ipv6Direct.ipv6Address"
                type="button"
                class="font-mono font-medium truncate hover:text-primary"
                @click="copyText(data.ipv6Direct.ipv6Address)"
              >
                {{ data.ipv6Direct.ipv6Address }}
              </button>
              <span v-else class="text-muted-foreground">
                {{ t("admin.ddns.notConfigured") }}
              </span>
            </div>
            <div v-if="data.ipv6Direct.targetDomain" class="flex items-center gap-2 text-xs">
              <span class="text-muted-foreground">域名</span>
              <span class="font-mono truncate">
                {{ data.ipv6Direct.targetDomain }}
              </span>
            </div>
          </div>
        </div>

        <!-- 网关 B: IPv4 Cloudflare -->
        <div
          class="rounded-xl border p-4"
          :class="
            data.ipv4Cloudflare.enabled
              ? 'border-blue-500/30 bg-blue-500/5'
              : 'border-muted bg-muted/30'
          "
        >
          <div class="mb-3 flex items-center gap-2">
            <div
              class="p-1.5 rounded-lg"
              :class="
                data.ipv4Cloudflare.enabled
                  ? 'bg-blue-500/15 text-blue-600 dark:text-blue-400'
                  : 'bg-muted text-muted-foreground'
              "
            >
              <Cloud class="h-4 w-4" />
            </div>
            <div class="min-w-0">
              <p class="text-sm font-medium truncate">
                {{ data.ipv4Cloudflare.label }}
              </p>
              <p class="text-[11px] text-muted-foreground">A → Cloudflare Tunnel</p>
            </div>
            <span
              class="ml-auto inline-flex h-2 w-2 rounded-full"
              :class="
                data.ipv4Cloudflare.enabled
                  ? 'bg-blue-500'
                  : 'bg-muted-foreground/30'
              "
            />
          </div>
          <p class="text-xs text-muted-foreground mb-2">
            {{ data.ipv4Cloudflare.description }}
          </p>
          <div class="space-y-1">
            <div v-if="data.ipv4Cloudflare.zoneName" class="flex items-center gap-2 text-xs">
              <span class="text-muted-foreground">Zone</span>
              <span class="font-mono truncate">
                {{ data.ipv4Cloudflare.zoneName }}
              </span>
            </div>
            <div v-if="data.ipv4Cloudflare.tunnelId" class="flex items-center gap-2 text-xs">
              <span class="text-muted-foreground">Tunnel</span>
              <span class="font-mono truncate">
                {{ data.ipv4Cloudflare.tunnelId }}
              </span>
            </div>
          </div>
        </div>
      </div>
    </CardContent>

    <CardContent v-else-if="loading" class="text-sm text-muted-foreground">
      {{ t("common.loading") }}...
    </CardContent>

    <CardContent v-else class="text-sm text-muted-foreground">
      {{ t("admin.ddns.statusLoadFailed") }}
    </CardContent>
  </Card>
</template>
