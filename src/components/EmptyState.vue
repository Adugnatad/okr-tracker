<template>
  <div class="empty">
    <div v-if="icon" class="empty__icon" :class="`fa fa-${icon}`"></div>
    <div v-if="heading" class="empty__heading" v-text="heading"></div>
    <div v-if="body" class="empty__body" v-text="body"></div>

    <div class="empty__actions">
      <slot></slot>
    </div>
  </div>
</template>

<script>
export default {
  name: 'EmptyState',

  props: {
    icon: {
      type: String,
      required: false,
      default: '',
    },
    heading: {
      type: String,
      required: false,
      default: '',
    },
    body: {
      type: String,
      required: false,
      default: '',
    },
  },
};
</script>

<style lang="scss" scoped>
.empty {
  display: flex;
  flex-direction: column;
  align-items: center;
  max-width: 30rem;
  margin: 1.5rem auto 1rem;
  padding: 1rem 1.5rem;
  color: var(--color-grey-400);
  text-align: center;
  border-radius: 3px;
}

.empty__icon {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 5rem;
  height: 5rem;
  margin-bottom: 2rem;
  color: var(--color-bg);
  font-size: 2.5rem;
  text-align: center;
  background: var(--color-grey-100);
  border-radius: 3.5rem;
}

.empty__heading {
  margin-bottom: 0.5rem;
  font-weight: 500;
  word-break: break-all;
}

.empty__body {
  color: var(--color-grey-300);
  word-break: break-all;
}

.empty__actions {
  margin-top: 1.5rem;
}
</style>
