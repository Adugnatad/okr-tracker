<template>
  <div v-if="activeKpi" class="container">
    <div class="widgets--left">
      <router-link class="btn widget__back-button" :to="{ name: 'ItemHome', params: { slug: activeItem.slug } }">
        {{ $t('general.back') }}
        <i class="fa fa-chevron-left"></i>
      </router-link>

      <div class="aside--left">
        <div class="widgets">
          <widget-mission-statement />
          <widget-team />
        </div>
      </div>
    </div>

    <div class="main">
      <div class="main__item">
        <h1 class="title-1">{{ activeKpi.name }}</h1>

        <p>{{ activeKpi.description }}</p>

        <div class="main-widgets">
          <div v-if="activeKpi.valid" class="main-widgets__current">
            <h3 class="main-widgets__title">
              {{ $t('kpi.currentValue') }}
            </h3>
            <div class="main-widgets__current--value">
              {{
                filteredProgress.length
                  ? formatKPIValue(filteredProgress[0].value)
                  : formatKPIValue(activeKpi.currentValue)
              }}
            </div>
          </div>

          <div class="main-widgets__graph">
            <h3 class="main-widgets__title">
              {{ $t('kpi.progress') }}
            </h3>

            <svg ref="graph" class="graph"></svg>
          </div>
        </div>

        <widgets-k-p-i-home class="aside--middle" :range="range" :progress="progress" @listen="handleChange" />

        <div class="widget__history">
          <h2 class="title-2">{{ $t('keyResultPage.history') }}</h2>

          <v-spinner v-if="isLoading"></v-spinner>

          <empty-state
            v-else-if="!filteredProgress.length || filteredProgress.length === 0"
            :icon="'history'"
            :heading="$t('empty.kpiProgress.heading')"
            :body="$t('empty.kpiProgress.body')"
          />

          <table v-else class="table">
            <thead>
              <tr>
                <th>{{ $t('keyResult.dateAndTime') }}</th>
                <th>{{ $t('keyResultPage.table.value') }}</th>
                <th v-if="hasEditRights"></th>
              </tr>
            </thead>
            <tbody></tbody>
            <tr v-for="{ timestamp, value, id } in limitedProgress" :key="id">
              <td>{{ dateTimeShort(timestamp.toDate()) }}</td>
              <td>{{ formatKPIValue(value) }}</td>
              <td v-if="hasEditRights" style="width: 1rem">
                <v-popover offset="16" placement="top" show="true">
                  <button class="btn btn--ter">
                    {{ $t('btn.delete') }}
                  </button>

                  <template slot="popover">
                    <button class="btn btn--ter btn--negative" @click="remove(id)">
                      {{ $t('btn.confirmDeleteProgress') }}
                    </button>
                  </template>
                </v-popover>
              </td>
            </tr>
          </table>
          <button
            v-if="filteredProgress.length > 10 && historyLimit !== null"
            class="btn btn--sec"
            style="align-self: center"
            @click="historyLimit = null"
          >
            {{ $t('btn.showMore') }}
          </button>
        </div>
      </div>
    </div>

    <widgets-k-p-i-home class="aside--right" :range="range" :progress="progress" @listen="handleChange" />
  </div>
</template>

<script>
import { mapGetters, mapState } from 'vuex';
import { extent } from 'd3-array';
import endOfDay from 'date-fns/endOfDay';
import { db } from '@/config/firebaseConfig';
import LineChart from '@/util/LineChart';
import { dateTimeShort, formatISOShort, numberLocale } from '@/util';
import kpiTypes from '@/config/kpiTypes';
import WidgetMissionStatement from '@/components/widgets/WidgetMissionStatement.vue';
import WidgetTeam from '@/components/widgets/WidgetTeam/WidgetTeam.vue';

export default {
  name: 'KpiHome',

  components: {
    WidgetsKPIHome: () => import('@/components/widgets/WidgetsKPIHome.vue'),
    EmptyState: () => import('@/components/EmptyState.vue'),
    WidgetMissionStatement,
    WidgetTeam,
  },

  data: () => ({
    progress: [],
    graph: null,
    range: '',
    startDate: null,
    endDate: null,
    filteredProgress: [],
    isFiltered: false,
    isLoading: false,
    historyLimit: 10,
  }),

  computed: {
    ...mapState(['activeKpi', 'activeItem', 'theme']),
    ...mapGetters(['hasEditRights']),

    limitedProgress() {
      return this.historyLimit ? this.filteredProgress.slice(0, this.historyLimit) : this.filteredProgress;
    },
  },

  watch: {
    activeKpi: {
      immediate: true,
      async handler({ id }) {
        this.isLoading = true;
        await this.$bind('progress', db.collection(`kpis/${id}/progress`).orderBy('timestamp', 'desc'));
        this.isLoading = false;
      },
    },
    progress() {
      this.filterProgress();
    },
  },

  mounted() {
    if (this.$refs.graph) {
      this.graph = new LineChart(this.$refs.graph, { colorMode: this.theme });
    }

    if (this.$route.query.startDate && this.$route.query.endDate) {
      this.startDate = this.$route.query.startDate;
      this.endDate = this.$route.query.endDate;
      this.range = `${this.startDate} til ${this.endDate}`;
    }
  },

  methods: {
    dateTimeShort,

    async remove(id) {
      try {
        await db.doc(`kpis/${this.activeKpi.id}/progress/${id}`).delete();
        this.$toasted.show(this.$t('toaster.delete.progression'));
      } catch {
        this.$toasted.error(this.$t('toaster.error.deleteProgress'));
      }
    },

    renderGraph() {
      const [startValue, targetValue] = extent(this.filteredProgress.map(({ value }) => value));
      const [startDate, endDate] = extent(this.filteredProgress.map(({ timestamp }) => timestamp));

      if (!this.graph || !startValue === undefined || !targetValue === undefined) return;
      this.graph.render({
        obj: {
          startValue,
          targetValue,
        },
        period: {
          endDate: this.isFiltered ? endDate : new Date(),
          startDate,
        },
        progressionList: this.filteredProgress,
        item: this.activeKpi,
        theme: this.theme,
      });
    },

    filterProgress() {
      if (!this.startDate && !this.endDate) {
        this.filteredProgress = this.progress;
      } else {
        this.filteredProgress = this.progress.filter(
          (a) => a.timestamp.toDate() > this.startDate && a.timestamp.toDate() < this.endDate
        );
      }
      this.renderGraph();
    },

    formatKPIValue(value) {
      if (kpiTypes[this.activeKpi.type].type === 'users') {
        return numberLocale.format(',')(value);
      }
      return kpiTypes[this.activeKpi.type].formatValue(value);
    },

    handleChange(range) {
      if (!range) {
        this.$router
          .push({
            name: 'KpiHome',
            params: { slug: this.$route.params.slug, kpiId: this.$route.params.kpiId },
          })
          .catch((err) => console.log(err));
        this.startDate = null;
        this.endDate = null;
        this.filteredProgress = this.progress;
        this.filterProgress();
        return;
      }
      const parts = range.split(' til ').map((d) => new Date(d));
      if (parts.length === 1) return;
      this.dirty = true;
      const [startDate, endDate] = parts;
      this.startDate = startDate;
      this.endDate = endOfDay(endDate);
      this.isFiltered = true;

      this.$router
        .push({
          name: 'KpiHome',
          params: { slug: this.$route.params.slug, kpiId: this.$route.params.kpiId },
          query: { startDate: formatISOShort(this.startDate), endDate: formatISOShort(this.endDate) },
        })
        .catch((err) => console.log(err));

      this.filterProgress();
    },
  },
};
</script>

<style lang="scss" scoped>
.history {
  margin: 2.5rem 0 1.5rem;
}

.main-widgets {
  display: flex;
  flex-direction: column;
  margin-top: 1.5rem;
  margin-bottom: 2rem;
}

.main-widgets__title {
  margin-bottom: 0.5rem;
  color: var(--color-text);
  font-weight: 500;
}

.main-widgets__current {
  align-self: flex-start;
  width: span(12);

  padding: 1rem;
  background: var(--color-secondary);
  box-shadow: 0 2px 3px rgba(black, 0.1);
}

.main-widgets__current--value {
  color: var(--color-primary);
  font-weight: 700;
  font-size: 50px;
  word-wrap: break-word;
}

.main-widgets__graph {
  width: span(12);

  margin-top: 0.5rem;
  padding: 1rem;
  background: white;
}
</style>
