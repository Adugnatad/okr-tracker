<template>
  <div v-if="markdown" class="container">
    <h1 class="title-1">{{ $t('help.title') }}</h1>
    <div class="help">
      <ul v-if="toc" class="toc help__toc">
        <h1 class="title-1">{{ $t('help.toc') }}</h1>
        <li v-for="levelOne in toc.children" :key="levelOne.id">
          <a :href="`#${levelOne.data.id}`">{{ levelOne.data.text }}</a>
          <ul v-if="levelOne.children">
            <li v-for="levelTwo in levelOne.children" :key="levelTwo.id">
              <a :href="`#${levelTwo.data.id}`">{{ levelTwo.data.text }}</a>
              <ul v-if="levelTwo.children">
                <li v-for="levelThree in levelTwo.children" :key="levelThree.id">
                  <a :href="`#${levelThree.data.id}`">{{ levelThree.data.text }}</a>
                </li>
              </ul>
            </li>
          </ul>
        </li>
      </ul>

      <div class="help__content md" v-html="markdown"></div>
    </div>
  </div>
</template>

<script>
import { marked } from 'marked';
import dompurify from 'dompurify';
import { tableOfContent as toc } from '@/util';
import i18n from '@/locale/i18n';

marked.setOptions({
  smartypants: true,
});

export default {
  name: 'Help',

  data: () => ({
    markdown: '',
    toc: [],
  }),

  metaInfo() {
    return {
      title: `${i18n.t('general.help')} | ${i18n.t('general.project')}`,
    };
  },

  async created() {
    fetch('./help.md').then(async (response) => {
      this.markdown = dompurify.sanitize(marked(await response.text()));
      this.toc = toc(this.markdown);
    });
  },
};
</script>

<style lang="scss" scoped>
.help {
  @media screen and (min-width: bp(l)) {
    display: grid;
    grid-gap: span(0, 1);
    grid-template-columns: repeat(10, 1fr);
  }
}

.help__toc {
  align-self: start;
  @media screen and (min-width: bp(l)) {
    grid-row: 1;
    grid-column: 7 / span 4;
  }
}

.help__content {
  @media screen and (min-width: bp(l)) {
    grid-row: 1;
    grid-column: 1 / span 5;
  }

  @media screen and (min-width: bp(xl)) {
    grid-row: 1;
    grid-column: 1 / span 4;
  }
}

::v-deep a[href*='thumbnail'] img {
  float: left;
  width: 6em;
  height: 6em;
  margin: 1rem;
  object-fit: cover;
  border: 1px solid rgba(black, 0.1);
}

.toc {
  margin: 2rem 0 5rem;
  padding: 1.5rem;
  font-size: 1rem;
  border: 1px solid var(--color-grey-100);

  li {
    padding: 0.25rem 0;
  }

  ul {
    padding-top: 0.25rem;
    padding-left: 2rem;
  }

  a {
    font-weight: 400 !important;
  }
}
</style>
