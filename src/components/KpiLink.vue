<template>
  <div>
    <div v-if="!data.created" class="kpi kpi--empty">
      <div class="kpi__name">{{ kpi.label }}</div>
      <i class="kpi__icon far" :class="kpi.icon" />
    </div>

    <router-link
      v-else
      v-tooltip="kpi.label"
      :to="{ name: 'KpiHome', params: { kpiId: data.id } }"
      class="kpi"
      :class="{ disabled: data.error }"
    >
      <div class="kpi__name">{{ data.name }}</div>
      <div class="kpi__value">
        <span v-if="data.error || !data.valid">–––</span>
        <span v-else>{{ formatKPIValue(data.currentValue) }}</span>
      </div>
      <i class="kpi__icon far" :class="kpi.icon" />
    </router-link>
  </div>
</template>

<script>
import kpiTypes from '@/config/kpiTypes';
import { numberLocale } from '@/util';

export default {
  name: 'KpiLink',

  props: {
    type: {
      type: String,
      required: true,
    },
    kpi: {
      type: Object,
      required: true,
    },
    data: {
      type: Object,
      required: false,
      default: () => ({}),
    },
  },

  methods: {
    formatKPIValue(value) {
      if (kpiTypes[this.type].type === 'users') {
        return numberLocale.format(',')(value);
      }
      return kpiTypes[this.type].formatValue(value);
    },
  },
};
</script>

<style lang="scss" scoped>
.kpi {
  position: relative;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  height: 8rem;
  padding: 1rem;
  overflow: hidden;
  color: var(--color-text);
  font-weight: 500;
  text-decoration: none;
  background: var(--color-secondary-light);

  &:hover {
    color: var(--color-text-secondary);
    background: var(--color-hover);
  }

  &.disabled {
    opacity: 0.6;
  }
}

.kpi--empty {
  color: rgba(var(--color-purple-rgb), 0.25);
  background: rgba(var(--color-grey-100-rgb), 0.5);

  &:hover {
    background: rgba(var(--color-grey-100-rgb), 0.4);
  }

  & > .kpi__icon {
    opacity: 0.25;
  }
}

.kpi__value {
  font-weight: 900;
  font-size: 3rem;
  line-height: 1em;
}

.kpi__icon {
  position: absolute;
  right: -1rem;
  bottom: -1rem;
  font-size: 8rem;
  opacity: 0.15;
}
</style>
