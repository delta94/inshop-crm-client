<template>
  <v-app id="inspire">
    <v-navigation-drawer
      v-model="drawer"
      :clipped="$vuetify.breakpoint.lgAndUp"
      app
      dark
      hide-overlay
      :src="bg"
    >
      <v-list dense>
        <template v-for="item in items">
          <v-list-group
            v-if="isGrantedItem(item)"
            :key="item.label"
            ref="listGroup"
            v-model="item.model"
            :prepend-icon="prependIcon(item)"
            :append-icon="appendIcon(item)"
            @click="listItemClick(item)"
          >
            <template v-slot:activator>
              <v-list-item>
                <v-list-item-content>
                  <v-list-item-title>
                    {{ $t(item.label) }}
                  </v-list-item-title>
                </v-list-item-content>
              </v-list-item>
            </template>

            <template v-for="(child, i) in item.children">
              <v-list-item
                v-if="isGrantedItem(child)"
                :key="i"
                :class="{
                  'active-item': routeName(child.route) === activeRoute
                }"
                @click="listItemClick(child)"
              >
                <v-list-item-action v-if="child.icon">
                  <v-icon right>
                    {{ child.icon }}
                  </v-icon>
                </v-list-item-action>
                <v-list-item-content>
                  <v-list-item-title>
                    {{ $t(child.label) }}
                  </v-list-item-title>
                </v-list-item-content>
              </v-list-item>
            </template>
          </v-list-group>
        </template>
      </v-list>
    </v-navigation-drawer>

    <v-app-bar
      :clipped-left="$vuetify.breakpoint.lgAndUp"
      app
      color="primary"
      dark
      :src="bg"
    >
      <v-toolbar-title style="width: 300px" class="ml-0">
        <v-app-bar-nav-icon @click.stop="drawer = !drawer"></v-app-bar-nav-icon>
        <span class="hidden-sm-and-down mx-2">Inshop CRM</span>
      </v-toolbar-title>

      <v-row class="flex-column">
        <v-form
          v-if="isGranted('ROLE_OTHER_SEARCH')"
          method="get"
          class="hidden-sm-and-down"
          @submit.prevent="search"
        >
          <v-text-field
            v-model="q"
            flat
            hide-details
            solo-inverted
            prepend-inner-icon="search"
            label="Search"
          ></v-text-field>
        </v-form>
      </v-row>

      <v-spacer></v-spacer>

      <span class="mx-2">{{ username }}</span>
      <v-btn icon @click="signOut">
        <v-icon>mdi-power</v-icon>
      </v-btn>
    </v-app-bar>

    <v-main>
      <router-view />
    </v-main>

    <is-footer></is-footer>

    <modal-not-done-tasks
      :tasks="tasks"
      :dialog="dialog"
      @dialog-close="dialog = false"
    ></modal-not-done-tasks>
  </v-app>
</template>

<script>
import fecha from 'fecha'
import bg from '@/assets/bg.png'
import ModalNotDoneTasks from './ModalNotDoneTasks'
import axios from '../../plugins/axios'

export default {
  name: 'MainSlot',
  components: { ModalNotDoneTasks },
  data() {
    return {
      bg,
      fecha,
      q: null,
      tasks: [],
      dialog: false,
      drawer: null,
      activeRoute: null,
      items: [
        {
          label: 'dashboard',
          icon: 'dashboard',
          route: 'Dashboard',
          role: 'ROLE_USER_DASHBOARD'
        },
        {
          label: 'calendar',
          icon: 'perm_contact_calendar',
          route: 'Calendar',
          role: 'ROLE_OTHER_CALENDAR'
        },
        {
          label: 'clients',
          icon: 'people',
          route: 'ClientList',
          role: 'ROLE_CLIENT_LIST'
        },
        {
          label: 'companies',
          icon: 'grade',
          route: 'CompanyList',
          role: 'ROLE_COMPANY_LIST'
        },
        {
          label: 'projects',
          icon: 'folder_open',
          route: 'ProjectList',
          role: 'ROLE_PROJECT_LIST'
        },
        {
          label: 'tasks',
          icon: 'access_time',
          route: 'TaskList',
          role: 'ROLE_TASK_LIST'
        },
        {
          label: 'documents',
          icon: 'turned_in',
          route: 'DocumentList',
          role: 'ROLE_DOCUMENT_LIST'
        },
        {
          label: 'contacts',
          icon: 'contacts',
          children: [
            {
              label: 'contacts',
              icon: 'local_phone',
              route: 'ContactList',
              role: 'ROLE_CONTACT_LIST'
            },
            {
              label: 'address',
              icon: 'my_location',
              route: 'AddressList',
              role: 'ROLE_ADDRESS_LIST'
            }
          ]
        },
        {
          label: 'product',
          icon: 'store',
          children: [
            {
              label: 'category',
              icon: 'format_align_justify',
              route: 'CategoryList',
              role: 'ROLE_CATEGORY_LIST'
            },
            {
              label: 'product',
              icon: 'format_align_justify',
              route: 'ProductList',
              role: 'ROLE_PRODUCT_LIST'
            }
          ]
        },
        {
          label: 'cms',
          icon: 'mode_edit',
          children: [
            {
              label: 'text',
              icon: 'text_fields',
              route: 'TextList',
              role: 'ROLE_TEXT_LIST'
            }
          ]
        },
        {
          label: 'order_header',
          icon: 'local_grocery_store',
          children: [
            {
              label: 'order_header',
              icon: 'assignment',
              route: 'OrderHeaderList',
              role: 'ROLE_ORDER_HEADER_LIST'
            },
            {
              label: 'order_status',
              icon: 'library_books',
              route: 'OrderStatusList',
              role: 'ROLE_ORDER_STATUS_LIST'
            },
            {
              label: 'order_line_status',
              icon: 'format_align_justify',
              route: 'OrderLineStatusList',
              role: 'ROLE_ORDER_LINE_STATUS_LIST'
            }
          ]
        },
        {
          label: 'invoice_header',
          icon: 'attach_money',
          children: [
            {
              label: 'invoice_header',
              icon: 'format_align_justify',
              route: 'InvoiceHeaderList',
              role: 'ROLE_INVOICE_HEADER_LIST'
            },
            {
              label: 'invoice_status',
              icon: 'format_align_justify',
              route: 'InvoiceStatusList',
              role: 'ROLE_INVOICE_STATUS_LIST'
            },
            {
              label: 'invoice_type',
              icon: 'format_align_justify',
              route: 'InvoiceTypeList',
              role: 'ROLE_INVOICE_TYPE_LIST'
            }
          ]
        },
        {
          label: 'dictionaries',
          icon: 'list',
          children: [
            {
              label: 'country',
              icon: 'flag',
              route: 'CountryList',
              role: 'ROLE_COUNTRY_LIST'
            },
            {
              label: 'language',
              icon: 'format_align_justify',
              route: 'LanguageList',
              role: 'ROLE_LANGUAGE_LIST'
            },
            {
              label: 'vat',
              icon: 'format_align_justify',
              route: 'VatList',
              role: 'ROLE_VAT_LIST'
            },
            {
              label: 'currency',
              icon: 'format_align_justify',
              route: 'CurrencyList',
              role: 'ROLE_CURRENCY_LIST'
            },
            {
              label: 'contact_type',
              icon: 'call_end',
              route: 'ContactTypeList',
              role: 'ROLE_CONTACT_TYPE_LIST'
            },
            {
              label: 'project_type',
              icon: 'work',
              route: 'ProjectTypeList',
              role: 'ROLE_PROJECT_TYPE_LIST'
            },
            {
              label: 'project_status',
              icon: 'web',
              route: 'ProjectStatusList',
              role: 'ROLE_PROJECT_STATUS_LIST'
            },
            {
              label: 'task_status',
              icon: 'web_asset',
              route: 'TaskStatusList',
              role: 'ROLE_TASK_STATUS_LIST'
            },
            {
              label: 'brand',
              icon: 'format_align_justify',
              route: 'BrandList',
              role: 'ROLE_BRAND_LIST'
            },
            {
              label: 'channel',
              icon: 'format_align_justify',
              route: 'ChannelList',
              role: 'ROLE_CHANNEL_LIST'
            },
            {
              label: 'label',
              icon: 'label',
              route: 'LabelList',
              role: 'ROLE_LABEL_LIST'
            },
            {
              label: 'paymentType',
              icon: 'format_align_justify',
              route: 'PaymentTypeList',
              role: 'ROLE_PAYMENT_TYPE_LIST'
            },
            {
              label: 'shipmentMethod',
              icon: 'format_align_justify',
              route: 'ShipmentMethodList',
              role: 'ROLE_SHIPMENT_METHOD_LIST'
            }
          ]
        },
        {
          label: 'users',
          icon: 'security',
          children: [
            {
              label: 'users',
              icon: 'account_box',
              route: 'UserList',
              role: 'ROLE_USER_LIST'
            },
            {
              label: 'groups',
              icon: 'people_outline',
              route: 'GroupList',
              role: 'ROLE_GROUP_LIST'
            }
          ]
        },
        {
          label: 'maintenance',
          icon: 'settings',
          children: [
            {
              label: 'history',
              icon: 'timelapse',
              route: 'HistoryList',
              role: 'ROLE_HISTORY_LIST'
            }
          ]
        }
      ]
    }
  },
  computed: {
    username() {
      const jwt = this.$store.getters['auth/jwtDecoded']

      return jwt ? jwt.name : null
    }
  },
  watch: {
    $route(to, from) {
      this.activeRoute = this.routeName(to.name)
      this.initActiveMenuItem()
    }
  },
  created() {
    this.activeRoute = this.routeName(this.$route.name)
  },
  mounted() {
    if (this.isGranted('ROLE_TASK_LIST')) {
      this.getTasks()
    }

    this.initActiveMenuItem()
  },
  methods: {
    initActiveMenuItem() {
      this.items.forEach(i => {
        if (i.children && i.children.length) {
          i.children.forEach(el => {
            if (this.routeName(el.route) === this.activeRoute) {
              i.model = true
            }
          })
        } else if (this.routeName(i.route) === this.activeRoute) {
          i.model = true
        }
      })
    },
    routeName(route) {
      return route ? route.replace(/(List|Show|Create|Update)/, '') : ''
    },
    prependIcon(item) {
      if (item.icon) {
        return item.icon
      }
    },
    appendIcon(item) {
      if (item.children && item.children.length) {
        return 'keyboard_arrow_down'
      }

      return null
    },
    listItemClick(item) {
      if (item.route) {
        this.$router.push({ name: item.route })
      }
    },
    isGrantedItem(item) {
      if (item.role) {
        return this.isGranted(item.role)
      }

      if (item.children) {
        return item.children.some(child => this.isGranted(child.role))
      }

      return false
    },
    search() {
      this.$router.push({ name: 'Search', query: { q: this.q } })
    },
    signOut() {
      this.$store.dispatch('auth/logout')
    },
    getTasks() {
      const url = `${process.env.VUE_APP_API_URL}/tasks/deadline`

      axios.get(url).then(response => {
        this.tasks = response.data['hydra:member']

        if (this.tasks.length) {
          this.dialog = true
        }
      })
    }
  }
}
</script>

<style lang="scss">
.v-navigation-drawer {
  .v-list {
    &-group--active {
      .v-list-group__items {
        background: #142430;
      }
      .v-list-group__header {
        background: #142430;
      }
    }
    .theme--dark {
      .v-icon {
        color: #fff !important;
      }
    }
  }
}
.active-item {
  position: relative;
  &::before {
    content: '';
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    opacity: 0.3 !important;
    background-color: #fafafa !important;
  }
}
</style>
