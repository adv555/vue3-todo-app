<script>
export default {
  name: 'Message',
  emits: ['hide'],
  props: {
    active: Boolean,
  },
  methods: {
    hide() {
      this.$emit('hide');
    },
  },
};
</script>

<template>
  <article class="message" :class="{ 'message--hidden': !active }">
    <div class="message-header">
      <slot name="header"></slot>
      <button class="delete" @click="hide"></button>
    </div>

    <div class="message-body">
      <slot :x="42">Message</slot>
    </div>
  </article>
</template>
