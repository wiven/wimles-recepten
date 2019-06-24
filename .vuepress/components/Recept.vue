<!--<template>
  <div>
    <Content/>
    <aside class="side">
      <p>
        <strong>Component File Name:</strong>
      </p>
      <p class="side-heading">{{ $options.fileName }}</p>
      <div class="frontmatter">
        <p>
          <strong>Frontmatter:</strong>
        </p>
        <p>{{ $frontmatter }}</p>
        <p>Categorie: {{ $frontmatter.category }}</p>
        <p>Aantal personen: {{ $frontmatter.people }}</p>
        <p>
          Bron:
          <a :href="$frontmatter.source" target="_blank">Link naar recept</a>
        </p>
        <p>Photo: {{ $frontmatter.photo }}</p>
      </div>
      <a href="/guide/#custom-layouts">← Go Back</a>
    </aside>
  </div>
</template>-->

<template>
  <div
    class="theme-container"
    :class="pageClasses"
    @touchstart="onTouchStart"
    @touchend="onTouchEnd"
  >
    <Navbar v-if="shouldShowNavbar" @toggle-sidebar="toggleSidebar"/>

    <div class="sidebar-mask" @click="toggleSidebar(false)"></div>

    <Sidebar :items="sidebarItems" @toggle-sidebar="toggleSidebar">
      <slot name="sidebar-top" slot="top"/>
      <slot name="sidebar-bottom" slot="bottom"/>
    </Sidebar>
    {{ $page.frontmatter.category }}
    <Home v-if="$page.frontmatter.home"/>

    <Page v-else :sidebar-items="sidebarItems">
      <p>Aantal personen: {{ $page.frontmatter.people }}</p>
      <slot name="page-top" slot="top"/>
      <slot name="page-bottom" slot="bottom"/>
    </Page>
  </div>
</template>

<script>
import Home from "@theme/components/Home.vue";
import Navbar from "@theme/components/Navbar.vue";
import Page from "@theme/components/Page.vue";
import Sidebar from "@theme/components/Sidebar.vue";
import { resolveSidebarItems } from "../util";
export default {
  components: { Home, Page, Sidebar, Navbar },
  data() {
    return {
      isSidebarOpen: false
    };
  },
  computed: {
    shouldShowNavbar() {
      const { themeConfig } = this.$site;
      const { frontmatter } = this.$page;
      if (frontmatter.navbar === false || themeConfig.navbar === false) {
        return false;
      }
      return (
        this.$title ||
        themeConfig.logo ||
        themeConfig.repo ||
        themeConfig.nav ||
        this.$themeLocaleConfig.nav
      );
    },
    shouldShowSidebar() {
      const { frontmatter } = this.$page;
      return (
        !frontmatter.home &&
        frontmatter.sidebar !== false &&
        this.sidebarItems.length
      );
    },
    sidebarItems() {
      return resolveSidebarItems(
        this.$page,
        this.$page.regularPath,
        this.$site,
        this.$localePath
      );
    },
    pageClasses() {
      const userPageClass = this.$page.frontmatter.pageClass;
      return [
        {
          "no-navbar": !this.shouldShowNavbar,
          "sidebar-open": this.isSidebarOpen,
          "no-sidebar": !this.shouldShowSidebar
        },
        userPageClass
      ];
    }
  },
  mounted() {
    this.$router.afterEach(() => {
      this.isSidebarOpen = false;
    });
    console.log(this.$page.frontmatter.people);
  },
  methods: {
    toggleSidebar(to) {
      this.isSidebarOpen = typeof to === "boolean" ? to : !this.isSidebarOpen;
    },
    // side swipe
    onTouchStart(e) {
      this.touchStart = {
        x: e.changedTouches[0].clientX,
        y: e.changedTouches[0].clientY
      };
    },
    onTouchEnd(e) {
      const dx = e.changedTouches[0].clientX - this.touchStart.x;
      const dy = e.changedTouches[0].clientY - this.touchStart.y;
      if (Math.abs(dx) > Math.abs(dy) && Math.abs(dx) > 40) {
        if (dx > 0 && this.touchStart.x <= 80) {
          this.toggleSidebar(true);
        } else {
          this.toggleSidebar(false);
        }
      }
    }
  }
};
</script>

<style src="prismjs/themes/prism-tomorrow.css"></style>