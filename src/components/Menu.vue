<template>
  <ion-menu side="start" content-id="main-content" type="overlay" :disabled="!isUserAuthenticated || $route.path === '/login'">
    <ion-header>
      <ion-toolbar>
        <ion-title>{{ $t("Job Manager") }}</ion-title>
      </ion-toolbar>
    </ion-header>

    <ion-content>
      <ion-list>
        <ion-menu-toggle auto-hide="false" v-for="(page, index) in getValidMenuItems(appPages)" :key="index">
          <ion-item
            v-if="page.url"
            button
            router-direction="root"
            :router-link="page.url"
            class="hydrated"
            :class="{ selected: selectedIndex === index }">
            <ion-icon slot="start" :ios="page.iosIcon" :md="page.mdIcon" />
            <ion-label>{{ $t(page.title) }}</ion-label>
          </ion-item>
          <ion-item-divider color="light" v-else>
            <ion-label color="medium">{{ $t(page.title) }}</ion-label>
          </ion-item-divider> 
        </ion-menu-toggle>
      </ion-list>
    </ion-content>
    <ion-footer>
      <ion-toolbar>
        <ion-item lines="none">
          <ion-label class="ion-text-wrap">
            <p class="overline">{{ instanceUrl }}</p>
          </ion-label>
          <ion-note slot="end">{{ userProfile?.userTimeZone }}</ion-note>
        </ion-item>
        <!-- showing product stores only when there are multiple options to choose from. -->
        <ion-item v-if="userProfile?.stores?.length > 2" lines="none">
          <ion-select interface="popover" :value="eComStore.productStoreId" @ionChange="setEComStore($event)">
            <ion-select-option v-for="store in (userProfile?.stores ? userProfile.stores : [])" :key="store.productStoreId" :value="store.productStoreId" >{{ store.storeName }}</ion-select-option>
          </ion-select>
        </ion-item>
        <ion-item v-else lines="none">
          <ion-label class="ion-text-wrap">
            {{ currentEComStore.storeName }}
          </ion-label>
        </ion-item>
        <!-- similarly, showing shopify configs only when there are multiple options to choose from 
        but if both product store and config have multiple options, then only option to choose
        product store will be visible -->
        <ion-item v-if="shopifyConfigs?.length > 1 && userProfile?.stores?.length < 3" lines="none">
          <ion-select interface="popover" :value="currentShopifyConfig?.shopifyConfigId" @ionChange="setShopifyConfig($event)">
            <ion-select-option v-for="shopifyConfig in shopifyConfigs" :key="shopifyConfig.shopifyConfigId" :value="shopifyConfig.shopifyConfigId" >{{ shopifyConfig.name ? shopifyConfig.name : shopifyConfig.shopifyConfigName }}</ion-select-option>
          </ion-select>
        </ion-item>
        <ion-item v-else lines="none">
          <ion-label class="ion-text-wrap">
           <p>{{ currentShopifyConfig.name ? currentShopifyConfig.name : currentShopifyConfig.shopifyConfigName }}</p> 
          </ion-label>
        </ion-item>
      </ion-toolbar>
    </ion-footer>
  </ion-menu>
</template>

<script lang="ts">
import {
  IonContent,
  IonFooter,
  IonHeader,
  IonIcon,
  IonItem,
  IonItemDivider,
  IonLabel,
  IonList,
  IonMenu,
  IonMenuToggle,
  IonNote,
  IonSelect,
  IonSelectOption,
  IonTitle,
  IonToolbar
} from "@ionic/vue";
import { computed, defineComponent } from "vue";
import { mapGetters } from "vuex";
import { albumsOutline, barChartOutline, calendarNumberOutline, compassOutline, iceCreamOutline, libraryOutline, pulseOutline, settingsOutline, sendOutline, shirtOutline, terminalOutline, ticketOutline } from "ionicons/icons";
import { useStore } from "@/store";
import emitter from "@/event-bus"
import { hasPermission } from "@/authorization";
import { useRouter } from "vue-router";

export default defineComponent({
  name: "Menu",
  components: {
    IonContent,
    IonFooter,
    IonHeader,
    IonIcon,
    IonItem,
    IonItemDivider,
    IonLabel,
    IonList,
    IonMenu,
    IonMenuToggle,
    IonNote,
    IonSelect,
    IonSelectOption,
    IonTitle,
    IonToolbar
  },
  computed: {
    ...mapGetters({
      isUserAuthenticated: 'user/isUserAuthenticated',
      currentFacility: 'user/getCurrentFacility',
      eComStore: 'user/getCurrentEComStore',
      instanceUrl: 'user/getInstanceUrl',
      userProfile: 'user/getUserProfile',
      currentShopifyConfig: 'user/getCurrentShopifyConfig',
      currentEComStore: 'user/getCurrentEComStore',
      shopifyConfigs: 'user/getShopifyConfigs',
    })
  },
  methods: {
    async setEComStore(event: CustomEvent) {
      if(this.userProfile && this.eComStore?.productStoreId !== event.detail.value) {
        await this.store.dispatch('user/setEcomStore', { 'productStoreId': event.detail.value })
        emitter.emit("productStoreOrConfigChanged")
      }
    },
    async setShopifyConfig(event: CustomEvent){
      await this.store.dispatch('user/setCurrentShopifyConfig', { 'shopifyConfigId': event.detail.value });
      emitter.emit("productStoreOrConfigChanged")
    }
  },
  setup() {
    const store = useStore();
    const router = useRouter();
    
    // Filtering array of app pages, retaining only those elements (pages) that have the necessary permissions for display.
    const getValidMenuItems = (appPages: any) => {
      return appPages.filter((appPage: any) => (!appPage.meta || !appPage.meta.permissionId) || hasPermission(appPage.meta.permissionId));
    }

    let appPages = [
      {
        title: "Pipeline",
        url: "/pipeline",
        iosIcon: pulseOutline,
        mdIcon: pulseOutline,
        dependsOnBaseURL: true,
        meta: {
          permissionId: "APP_PIPELINE_VIEW"
        }
      },
      {
        title: "Initial load",
        url: "/initial-load",
        iosIcon: iceCreamOutline,
        mdIcon: iceCreamOutline,
        dependsOnBaseURL: false,
        meta: {
          permissionId: "APP_INITIAL_LOAD_VIEW"
        }
      },
      {
        title: "Pre-order",
        url: "/pre-order",
        iosIcon: calendarNumberOutline,
        mdIcon: calendarNumberOutline,
        dependsOnBaseURL: false,
        meta: {
          permissionId: "APP_PREORDER_VIEW"
        }
      },
      {
        title: "Orders",
        url: "/orders",
        iosIcon: ticketOutline,
        mdIcon: ticketOutline,
        dependsOnBaseURL: false,
        meta: {
          permissionId: "APP_ORDERS_VIEW"
        }
      },
      {
        title: "Brokering",
        url: "/brokering",
        iosIcon: compassOutline,
        mdIcon: compassOutline,
        dependsOnBaseURL: false,
        meta: {
          permissionId: "APP_BROKERING_VIEW"
        }
      },
      {
        title: "Fulfillment",
        url: "/fulfillment",
        iosIcon: sendOutline,
        mdIcon: sendOutline,
        dependsOnBaseURL: false,
        meta: {
          permissionId: "APP_FULFILLMENT_VIEW"
        }
      },
      {
        title: "Inventory",
        url: "/inventory",
        iosIcon: albumsOutline,
        mdIcon: albumsOutline,
        dependsOnBaseURL: false,
        meta: {
          permissionId: "APP_INVENTORY_VIEW"
        }
      },
      {
        title: "Products",
        url: "/product",
        iosIcon: shirtOutline,
        mdIcon: shirtOutline,
        dependsOnBaseURL: false,
        meta: {
          permissionId: "APP_PRODUCT_VIEW"
        }
      },
      {
        title: "Reports",
        url: "/reports",
        iosIcon: barChartOutline,
        mdIcon: barChartOutline,
        dependsOnBaseURL: false,
        meta: {
          permissionId: "APP_REPORT_VIEW"
        }
      },
      {
        title: "Miscellaneous",
        url: "/miscellaneous",
        iosIcon: libraryOutline,
        mdIcon: libraryOutline,
        dependsOnBaseURL: false,
        meta: {
          permissionId: "APP_MISC_VIEW"
        }
      },
      {
        title: "Bulk editor",
        meta: {
          permissionId: "APP_BULK_EDITOR_VIEW"
        }
      },
      {
        title: "Schedule in bulk",
        url: "/bulk-editor",
        iosIcon: terminalOutline,
        mdIcon: terminalOutline,
        dependsOnBaseURL: false,
        meta: {
          permissionId: "APP_BULK_EDITOR_VIEW"
        }
      },
      {
        title: "Settings",
        url: "/settings",
        iosIcon: settingsOutline,
        mdIcon: settingsOutline,
        dependsOnBaseURL: true
      }
    ] as any;
    if (process.env.VUE_APP_BASE_URL) {
      appPages = appPages.filter((page : any) => page.dependsOnBaseURL);
    }

    const selectedIndex = computed(() => {
      const path = router.currentRoute.value.path
      return getValidMenuItems(appPages).findIndex((screen : any) => screen.url === path || screen.childRoutes?.includes(path))
    })

    return {
      albumsOutline,
      appPages,
      compassOutline,
      barChartOutline,
      calendarNumberOutline,
      getValidMenuItems,
      hasPermission,
      iceCreamOutline,
      libraryOutline,
      pulseOutline,
      selectedIndex,
      settingsOutline,
      sendOutline,
      shirtOutline,
      store,
      terminalOutline,
      ticketOutline,
    };
  }
});
</script>

<style scoped>
ion-menu.md ion-item.selected ion-icon {
  color: var(--ion-color-secondary);
}
ion-menu.ios ion-item.selected ion-icon {
  color: var(--ion-color-secondary);
}
ion-item.selected {
  --color: var(--ion-color-secondary);
}
</style>
