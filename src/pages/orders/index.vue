<template>
  <q-card>
    <q-toolbar>
      <div v-if="$route.path!=='/orders/view'" class="col-auto">
        <q-btn flat dense icon="arrow_back" v-close-popup />
      </div>
      <q-toolbar-title>{{$t('route.orders')}}</q-toolbar-title>
      <q-btn icon="add" flat round dense color="blue" @click="isDialogAdd=!isDialogAdd" />
      <q-btn icon="filter_list" flat round dense color="teal">
        <q-tooltip>{{$t('global.filter')}}</q-tooltip>
        <q-menu v-model="isFilter" class="q-pa-md" style="min-width:356px">
          <!-- <div class="q-pa-md"> -->
          <div class="row">
            <div class="col-12">
              <q-input v-model="pagination.filter" :dense="$store.getters.dense.input" debounce="500" :placeholder="$t('global.search')"
                       @update:model-value="onFilter">
                <template v-slot:append>
                  <q-icon v-if="pagination.filter===''" name="search" />
                  <q-icon v-else name="clear" class="cursor-pointer" @click="onFilter('')" />
                </template>
              </q-input>
            </div>
          </div>
          <div class="row">
            <div class="col-12">
              <q-input :model-value="pagination.date?$moment(pagination.date).format($store.getters.format.date):''"
                       :dense="$store.getters.dense.input"
                       readonly :label="$t('global.createdAt')" :hint="`${$t('global.format')}: ${$store.getters.format.date}`">
                <template v-slot:append>
                  <q-icon name="event" class="cursor-pointer">
                    <q-popup-proxy ref="refFilterDate" transition-show="scale" transition-hide="scale">
                      <q-date v-model="pagination.date" today-btn mask="YYYY-MM-DD" @update:model-value="()=>$refs.refFilterDate.hide()" />
                    </q-popup-proxy>
                  </q-icon>
                </template>
              </q-input>
            </div>
          </div>
          <div class="row">
            <div class="col-12">
              <q-option-group name="accepted_genres" v-model="pagination.flag" type="checkbox" color="primary" inline
                              :options="[{label:$t('global.cancelled'),value:0},{label:$t('global.processing'),value:1},{label:$t('global.confirmed'),value:2}]"
                              @update:model-value="onChangeFlag" />
              <!-- <q-toggle v-model="pagination.flag" left-label :label="pagination.flag?$t('global.working'):$t('global.cancelled')"
                        :false-value="0" :true-value="1" @update:model-value="onChangeFlag" /> -->
            </div>
          </div>
        </q-menu>
      </q-btn>
    </q-toolbar>
    <q-card-section class="q-pt-none q-pb-none">
      <q-list separator id="scroll-items" class="scroll" style="height:calc(100vh - 99px)">
        <q-infinite-scroll ref="refScrollTarget" @load="onScrollLoad" :offset="250">
          <tm-swipeitem v-for="(e,i) in rows" :key="i" leftValue="max" rightValue="111" v-touch-hold.mouse="()=>{onTouchHold(e)}">
            <template v-if="e.flag===1" v-slot:right>
              <!-- <q-btn-dropdown flat round dense color="indigo" icon="print" @click="onLoadDetails(e)">
                <q-list :dense="$store.getters.dense.form">
                  <q-item clickable v-close-popup @click="onPrinting({value:e,type:'details'})">
                    <q-item-section>
                      <q-item-label>
                        {{$t('product.printDetails')}}
                      </q-item-label>
                    </q-item-section>
                  </q-item>
                  <q-item clickable v-close-popup @click="onPrinting({value:e,type:'total'})">
                    <q-item-section>
                      <q-item-label>
                        {{$t('product.printTotal')}}
                      </q-item-label>
                    </q-item-section>
                  </q-item>
                </q-list>
              </q-btn-dropdown> -->
              <!-- <q-btn flat round dense icon="info" color="light-green" :size="$store.getters.dense.table?'sm':'md'" @click="onDetails(e)">
                <q-tooltip>{{$t('global.details')}}</q-tooltip>
              </q-btn> -->
              <q-btn no-caps class="q-btn--square" @click="onConfirmBill(e)">
                <q-icon name="check" color="blue" size="18px" />
              </q-btn>
              <q-btn no-caps class="q-btn--square" @click="onCancelBill(e)">
                <q-icon name="clear" color="negative" size="18px" />
              </q-btn>
            </template>
            <template v-slot:left>
              <q-item-section class="q-pl-lg q-pr-lg">
                <q-item-label caption lines="1">
                  <span>{{$t('warehouse.taxes')}}: </span>
                  <span class="text-red">
                    {{parseInt(e.prices*0.1).NumberFormat($store.getters.language)}}
                    <q-badge color="red" transparent>{{$store.getters.unitPrice}}</q-badge>
                  </span>
                </q-item-label>
                <q-item-label caption lines="1">
                  <span>{{$t('warehouse.totalize')}}: </span>
                  <span class="text-red">
                    {{parseInt(e.prices*1.1).NumberFormat($store.getters.language)}}
                    <q-badge color="red" transparent>{{$store.getters.unitPrice}}</q-badge>
                  </span>
                </q-item-label>
                <q-item-label caption lines="1">
                  <span>{{$t('warehouse.products')}}: </span>
                  <span class="text-blue">{{e.products}}</span>
                </q-item-label>
              </q-item-section>
            </template>
            <q-separator v-if="i>0" />
            <q-item>
              <q-item-section @click="onDetails(e)">
                <q-item-label>{{e.code}}</q-item-label>
                <q-item-label caption lines="1">
                  <span>{{$t('warehouse.prices')}}: </span>
                  <span class="text-red">
                    {{parseInt(e.prices).NumberFormat($store.getters.language)}}
                    <q-badge color="red" transparent>{{$store.getters.unitPrice}}</q-badge>
                  </span>
                </q-item-label>
                <q-item-label caption lines="1">
                  <span>{{$t('warehouse.quantities')}}: </span>
                  <span class="text-green">{{parseInt(e.quantities).NumberFormat($store.getters.language)}}</span>
                </q-item-label>
                <q-item-label caption lines="1">
                  <span>{{$t('global.createdAt')}}: </span>
                  <span class="text-blue">
                    {{e.createdAt?$moment(e.createdAt).format(`${$store.getters.format.date} HH:mm`):''}}
                  </span>
                </q-item-label>
              </q-item-section>
              <q-item-section side>
                <q-icon v-if="e.flag===0" name="cancel" color="red">
                  <q-popup-proxy>
                    <q-banner>
                      <span class="text-red">{{$t('warehouse.msgBillCanceled')}}</span>
                    </q-banner>
                  </q-popup-proxy>
                </q-icon>
                <q-icon v-else-if="e.flag===1" name="check_circle" @click="onConfirmBill(e)">
                </q-icon>
                <q-icon v-else name="check_circle" color="blue">
                  <q-popup-proxy>
                    <q-banner>
                      <span class="text-blue">{{$t('warehouse.msgBillConfirmed')}}</span>
                    </q-banner>
                  </q-popup-proxy>
                </q-icon>
              </q-item-section>
            </q-item>
          </tm-swipeitem>
          <template v-slot:loading>
            <div class="row justify-center q-my-md">
              <q-spinner-dots color="primary" size="40px" />
            </div>
          </template>
        </q-infinite-scroll>
      </q-list>
    </q-card-section>
  </q-card>

  <!-- Printer Details -->
  <tm-printer ref="refPrinterDetails" hidden :title="printer.title">
    <template v-slot:content>
      <view-details :title="$t('product.importBallotDetails')" :rows="printer.details"
                    :total="printer.total" color="secondary" :label="$t('product.printDetails')"
                    :labelDate="$t('product.importDate')" />
    </template>
  </tm-printer>
  <!-- Printer Total -->
  <tm-printer ref="refPrinterTotal" hidden :title="printer.title">
    <template v-slot:content>
      <view-total :title="$t('product.importBallotTotal')" :total="printer.total" color="blue"
                  :label="$t('product.printTotal')" :labelDate="$t('product.importDate')" />
    </template>
  </tm-printer>
  <!-- Details dialog -->
  <q-dialog v-model="isDialogDetails" maximized>
    <q-card flat :style="{minWidth:'60%'}">
      <q-toolbar>
        <div class="col-auto">
          <q-btn flat dense icon="arrow_back" v-close-popup></q-btn>
        </div>
        <span>{{$t('product.exportBallotDetails')}}</span>
      </q-toolbar>
      <q-separator />
      <!-- <q-scroll-area style="height:calc(100vh - 180px)"> -->
      <view-details :rows="printer.details" :total="printer.total" color="secondary" :label="$t('product.printDetails')"
                    :labelDate="$t('product.exportDate')" hidden-boot />
      <!-- </q-scroll-area> -->
    </q-card>
  </q-dialog>
  <!-- Dialog Add -->
  <q-dialog v-model="isDialogAdd" maximized>
    <add-item />
  </q-dialog>
  <!-- Dialog Actions -->
  <q-dialog v-model="isDialogTouchHold" position="bottom">
    <q-card class="full-width" style="border-radius:initial">
      <q-card-section class="q-pa-none">
        <q-list separator>
          <q-item clickable v-ripple class="text-center" @click="()=>{isDialogTouchHold=!isDialogTouchHold;onEdit()}">
            <q-item-section>{{$t('global.edit')}}</q-item-section>
          </q-item>
          <q-item clickable v-ripple class="text-center" @click="()=>{isDialogTouchHold=!isDialogTouchHold;onTrash()}">
            <q-item-section>
              <q-item-label v-if="pagination.flag" class="text-red">{{$t('global.trash')}}</q-item-label>
              <q-item-label v-else class="text-warning">{{$t('global.recover')}}</q-item-label>
            </q-item-section>
          </q-item>
        </q-list>
        <q-separator size="3px" />
        <q-list>
          <q-item clickable v-ripple class="text-center" v-close-popup>
            <q-item-section>
              <q-item-label>{{$t('global.cancel')}}</q-item-label>
            </q-item-section>
          </q-item>
        </q-list>
      </q-card-section>
    </q-card>
  </q-dialog>
</template>

<script>
import { defineComponent, defineAsyncComponent, ref, computed } from "vue"
import { useStore } from 'vuex'
import { useQuasar } from 'quasar'
import { useI18n } from 'vue-i18n'
export default defineComponent({
  name: "OrdersIndex",
  components: {
    tmSwipeitem: defineAsyncComponent(() => import('components/tm-swipe-item/index.vue')),
    viewDetails: defineAsyncComponent(() => import('components/view-details/index.vue')),
    viewTotal: defineAsyncComponent(() => import('components/view-total/index.vue')),
    tmPrinter: defineAsyncComponent(() => import('components/tm-printer/index.vue')),
    addItem: defineAsyncComponent(() => import('./add.vue')),
  },
  setup () {
    const $store = useStore()
    const $q = useQuasar()
    const { t } = useI18n({ useScope: 'global' })
    const isDialogAdd = ref(false)
    const isDialogDetails = ref(false)
    const isDialogTouchHold = ref(false)
    const refScrollTarget = ref(null)
    const isFilter = ref(false)
    const selected = ref([])
    const categories = ref([])
    const refPrinterDetails = ref(null)
    const refPrinterTotal = ref(null)
    const printer = ref({ total: {}, details: [], title: '' })
    const pagination = ref({
      filter: '',
      sortBy: 'createdAt',
      descending: true,
      page: 1,
      rowsPerPage: 10,
      rowsNumber: 1,
      categories: null,
      flag: [1, 2],
      date: Date.now()
    })
    const data = ref([])
    const rows = computed(() => JSON.parse(JSON.stringify(data.value)))
    const onFetch = (opt) => {
      const { page, rowsPerPage, sortBy, descending, categories } = opt.pagination
      if (opt.scroll) pagination.value.page = page
      else {
        if (refScrollTarget.value) {
          document.getElementById('scroll-items').scroll(0, 0)
          refScrollTarget.value.reset()
        }
        pagination.value.page = 1
      }
      pagination.value.rowsPerPage = rowsPerPage
      pagination.value.sortBy = sortBy
      pagination.value.categories = categories
      pagination.value.descending = descending
      return $store.dispatch('exports/get', opt.pagination).then(x => {
        pagination.value.rowsNumber = x.rowsNumber
        pagination.value.totalPage = Math.ceil(pagination.value.rowsNumber / pagination.value.rowsPerPage)
        // data.value = data.value.concat(x.data)
        return x.data
      })
    }
    if ($store.state.auth.token) {
      onFetch({ pagination: pagination.value }).then(x => data.value = x)
    }
    const LoadDetails = (id) => {
      $store.dispatch('exports/getSub', { key: id }).then((x) => {
        printer.value.details = x.data
      })
    }
    return {
      rows, selected, categories, pagination, isDialogDetails, isDialogTouchHold, isDialogAdd, refScrollTarget, isFilter,
      onFetch, refPrinterDetails, refPrinterTotal, printer,
      onFilter: (val) => {
        pagination.value.filter = val
        onFetch({ pagination: pagination.value }).then(x => { data.value = x })
      },
      onChangeFlag: (val) => {
        isFilter.value = false
        selected.value = []
        pagination.value.flag = val
        onFetch({ pagination: pagination.value }).then(x => data.value = x)
      },
      onDetails (val) {
        LoadDetails(val._id)
        printer.value.total = val
        isDialogDetails.value = true
      },
      onLoadDetails (val) {
        LoadDetails(val._id)
      },
      onPrinting ({ value, type }) {
        printer.value.total = value
        if (type === 'details') {
          printer.value.title = `${t('product.importBallotDetails')}-${printer.value.total._id}`
          refPrinterDetails.value.onPrinting()
        } else if (type === 'total') {
          printer.value.title = `${t('product.importBallotTotal')}-${printer.value.total._id}`
          refPrinterTotal.value.onPrinting()
        }
      },
      onConfirmBill (val) {
        $q.dialog({
          title: t('messageBox.confirm'),
          message: t('warehouse.msgBillConfirm'),
          cancel: true,
          ok: {
            label: t('global.accept'),
            flat: true,
            color: 'primary',
            noCaps: true
          },
          cancel: {
            label: t('global.cancel'),
            flat: true,
            color: 'blue-grey',
            noCaps: true
          }
        }).onOk(() => {
          $store.dispatch('exports/patch', { id: [val._id] }).then((x) => {
            if (x && x.success && x.success.length) val.flag = 2
          })
        })
      },
      onCancelBill (val) {
        $q.dialog({
          title: t('messageBox.warning'),
          message: t('warehouse.msgBillCancel') + '<br/><br/><i>' + t('warehouse.msgBillCancelNote') + '</i>',
          cancel: true,
          html: true,
          ok: {
            label: t('global.accept'),
            flat: true,
            color: 'primary',
            noCaps: true
          },
          cancel: {
            label: t('global.cancel'),
            flat: true,
            color: 'blue-grey',
            noCaps: true
          }
        }).onOk(() => {
          $store.dispatch('exports/put', { id: [val._id] }).then((x) => {
            if (x && x.success && x.success.length) val.flag = 0
          })
        })
      },
      onScrollLoad (index, done) {
        pagination.value.page = index + 1
        if (pagination.value.page <= pagination.value.totalPage)
          onFetch({ pagination: pagination.value, scroll: true }).then(x => {
            data.value = data.value.concat(x)
            done()
          })
        else done()
      },
      onTouchHold (val) {
        if (val) selected.value = [val]
        isDialogTouchHold.value = !isDialogTouchHold.value
      }
    }
  }
})
</script>
