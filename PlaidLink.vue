<template>
    <div @click="handleClick" :style="{ display: 'inline' }">
        <slot></slot>
    </div>
</template>

<script setup lang="ts">
import { ref, onMounted } from "vue";

const props = defineProps({
    clientName: String,
    env: {
        type: String,
        required: true,
        default: "sandbox",
        validator: (value: string) => ["sandbox", "development", "production"].includes(value),
    },
    isWebview: Boolean,
    token: String,
    public_key: String,
    product: Array,
    receivedRedirectUri: String,
    webhook: String,
    onLoad: Function,
    onSuccess: {
        type: Function,
        required: true,
    },
    onExit: Function,
    onEvent: Function,
});

const plaid = ref<any>(null);
const linkHandler = ref<any>(null);

const openLink = () => {
    if (plaid.value != null) {
        if (linkHandler.value != null) {
            linkHandler.value.destroy();
            linkHandler.value = null;
        }

        const options: any = {
            clientName: props.clientName,
            env: props.env,
            isWebview: props.isWebview,
            onLoad: () => props.onLoad && props.onLoad(),
            onSuccess: (public_token: string, metadata: any) => props.onSuccess(public_token, metadata),
            onExit: (err: any, metadata: any) => props.onExit && props.onExit(err, metadata),
            onEvent: (eventName: string, metadata: any) => props.onEvent && props.onEvent(eventName, metadata),
            ...(props.public_key ? { key: props.public_key } : {}),
            ...(props.token ? { token: props.token } : {}),
            ...(props.product ? { product: props.product } : {}),
            ...(props.receivedRedirectUri ? { receivedRedirectUri: props.receivedRedirectUri } : {}),
            ...(props.webhook ? { webhook: props.webhook } : {}),
        };

        linkHandler.value = plaid.value.create(options);
        linkHandler.value.open();
    }
};

const handleClick = (e: Event) => {
    e.preventDefault();
    openLink();
};

onMounted(() => {
    if (window.Plaid) {
        plaid.value = window.Plaid;
        return;
    }
    const linkScript = document.createElement("script");
    linkScript.async = true;
    linkScript.src = "https://cdn.plaid.com/link/v2/stable/link-initialize.js";
    document.head.appendChild(linkScript);
    linkScript.onload = () => {
        plaid.value = window.Plaid;
    };
});

defineExpose({ openLink });
</script>
