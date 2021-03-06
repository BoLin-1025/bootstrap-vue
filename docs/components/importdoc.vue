<template>
  <section
    v-if="components.length > 0 || directives.length > 0"
    class="bd-content"
  >
    <template v-if="components.length > 0">
      <article>
        <anchored-heading id="importing-individual-components" level="3">
          Importing individual components
        </anchored-heading>

        <b-table
          :items="componentImports"
          class="bv-docs-table"
          responsive="sm"
          head-variant="default"
          bordered
          striped
        >
          <template slot="component" slot-scope="{ value }">
            <code class="text-nowrap">{{ value }}</code>
          </template>
          <template slot="importPath" slot-scope="{ value }">
            <code class="text-nowrap">{{ value }}</code>
          </template>
        </b-table>

        <p><strong>Example:</strong></p>

        <pre class="hljs js text-monospace p-2">{{ componentImportCode }}</pre>
      </article>
    </template>

    <template v-if="directives.length > 0">
      <article>
        <anchored-heading id="importing-individual-directives" level="3">
          Importing individual directives
        </anchored-heading>

        <b-table
          :items="directiveImports"
          class="bv-docs-table"
          responsive="sm"
          head-variant="default"
          bordered
          striped
        >
          <template slot="directive" slot-scope="{ value }">
            <code class="text-nowrap">{{ value }}</code>
          </template>
          <template slot="importPath" slot-scope="{ value }">
            <code class="text-nowrap">{{ value }}</code>
          </template>
        </b-table>

        <p><strong>Example:</strong></p>

        <pre class="hljs js text-monospace p-2">{{ directiveImportCode }}</pre>
      </article>
    </template>

    <article>
      <anchored-heading id="importing-as-a-plugin" level="3">
        Importing as a Vue.js plugin
      </anchored-heading>

      <p v-if="isComponentRoute">
        This plugin includes all of the above listed individual
        components<span v-if="directives.length"> and directives</span>.
        Plugins also include any component aliases.
      </p>
      <p v-else>
        This plugin includes all of the above listed individual directives.
      </p>

      <pre class="hljs js text-monospace p-2">{{ pluginImportCode }}</pre>

      <template v-if="meta.plugins && meta.plugins.length > 0">
        <p>This plugin also automatically includes the following plugins:</p>
        <ul>
          <li v-for="plugin in meta.plugins" :key="plugin"><code>{{ plugin }}</code></li>
        </ul>
      </template>
    </article>
  </section>
</template>

<script>
import hljs from 'highlight.js'
import kebabCase from 'lodash/kebabCase'
import startCase from 'lodash/startCase'
import AnchoredHeading from './anchored-heading'

export default {
  name: 'BDVImportdoc',
  components: { AnchoredHeading },
  props: {
    meta: {}
  },
  computed: {
    isComponentRoute() {
      return this.$route.name === 'docs-components-slug'
    },
    pluginDir() {
      return this.$route.params.slug
    },
    pluginName() {
      return startCase(this.pluginDir).replace(/\s+/g, '')
    },
    componentImports() {
      return this.components.map(c => {
        return {
          component: this.componentTag(c),
          importPath: this.componentPath(c)
        }
      })
    },
    directiveImports() {
      return this.directives.map(d => {
        return {
          directive: this.directiveAttr(d),
          importPath: this.directivePath(d)
        }
      })
    },
    components() {
      let subcomponents = []
      if (this.meta.components) {
        // We just want the sub-component name
        subcomponents = this.meta.components.map(m => m.component)
      }
      return [].concat(this.meta.component, subcomponents).filter(c => c)
    },
    directives() {
      return [].concat(this.meta.directive, this.meta.directives).filter(d => d)
    },
    componentImportCode() {
      const firstComponent = this.components[0]
      const firstComponentImport = this.componentImports[0]
      return [
        `import ${firstComponent} from '${firstComponentImport.importPath}'`,
        `Vue.component('${this.componentName(firstComponent)}', ${firstComponent})`
      ].join('\n')
    },
    directiveImportCode() {
      const firstDirective = this.directives[0]
      const firstDirectiveImport = this.directiveImports[0]
      return [
        "// Note: Vue automatically prefixes the directive name with 'v-'",
        `import ${firstDirective} from '${firstDirectiveImport.importPath}'`,
        `Vue.directive('${this.directiveName(firstDirective)}', ${firstDirective})`
      ].join('\n')
    },
    pluginImportCode() {
      const pluginLocation = this.isComponentRoute ? 'components' : 'directives'
      // This also works for importing plugins, but may not tree shake as well
      // return [
      //   `import { ${this.pluginName} } from 'bootstrap-vue/es/${pluginLocation}'`,
      //   `Vue.use(${this.pluginName})`
      // ].join('\n')
      return [
        `import ${this.pluginName} from 'bootstrap-vue/es/${pluginLocation}/${this.pluginDir}'`,
        `Vue.use(${this.pluginName})`
      ].join('\n')
    }
  },
  mounted() {
    // Highlight code blocks
    ;[...this.$el.querySelectorAll('pre.hljs')].forEach(pre => {
      hljs.highlightBlock(pre)
    })
  },
  methods: {
    componentName(component) {
      return kebabCase(component)
    },
    componentTag(component) {
      return `<${this.componentName(component)}>`
    },
    componentPath(component) {
      const componentName = this.componentName(component).replace(/^b-/, '')
      return `bootstrap-vue/es/components/${this.pluginDir}/${componentName}`
    },
    directiveName(directive) {
      return kebabCase(directive).replace(/^v-/, '')
    },
    directiveAttr(directive) {
      return kebabCase(directive)
    },
    directivePath(directive) {
      const directiveName = this.directiveName(directive).replace(/^b-/, '')
      return `bootstrap-vue/es/directives/${directiveName}/${directiveName}`
    }
  }
}
</script>
