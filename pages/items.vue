<template>
  <div>
    <div class="row">
      <div class="col-12">
        <alert :type="'alert-info'">Test Banner/Alert</alert>
        <div v-for="item in notices">
          <alert :type="'alert-info'" v-html="item.message">
          </alert>
        </div>
      </div>
    </div>
    <div class="row mw-100 mr-0">
      <div class="col-12 mr-0 pr-0">
        <su-dashboard-chart></su-dashboard-chart>
      </div>
    </div>
    <div class="row">
      <!-- Double mt-4 because the above boxes have weird sizing issues. Oh well... -->
      <div class="col-12 mt-4"><h3>Store Products</h3></div>
    </div>
    <div class="row">
      <div class="col-12 col-lg-4">
        <su-multiline-info-box title="Average Item Price" :data="aip">
        </su-multiline-info-box>
      </div>
      <div class="col-12 col-lg-4">
        <su-multiline-info-box title="Items per Order" :data="ipo">
        </su-multiline-info-box>
      </div>
      <div class="col-12 col-lg-4">
        <su-multiline-info-box title="Inventory Value" :data="inv">
        </su-multiline-info-box>
      </div>

    </div>
    <div class="row mt-4">
      <div class="col-12 col-lg-12">
        <su-list-box title="Top Selling Items" :columns="topItemsCol" :data="topItems">
        </su-list-box>
      </div>
      <!--<div class="col-12 col-lg-6">
        <su-list-box title="Top Selling Collections" :columns="topItemsCol" :data="topCollections">
        </su-list-box>
      </div>-->
    </div>
  </div>
</template>

<script>
import Alert from '../components/Alert'
import SuDashboardChart from "@/components/SuDashboardChart";
import SuMultilineInfoBox from "@/components/SuMultilineInfoBox";
import SuListBox from "@/components/SuListBox";
import {API_URL} from "@/config";

export default {
  name: 'Items',
  components: {SuListBox, SuMultilineInfoBox, SuDashboardChart, Alert },
  layout: 'dashboard',
  transition: 'fade',
  head() {
    return {
      title: 'Shoptrac | Items'
    }
  },
  async mounted() {
    await this.$axios
        .$get(`${API_URL}/items`, {
          headers: {
            Authorization: localStorage.getItem('token')
          }
        })
        .then((res) => {
          this.populate(res);
        })
        .catch((res) => {
        })
  },
  data() {
    return {
      notices: [],
      aip: {},
      ipo: {},
      inv: {},

      topItemsCol: ["Item Name", "Quantity Sold", "% of Item Sales", "Amount Sold ($)"],

      topItems: [],
      //topCollections: {},
    }
  },
  methods: {
    formatNumber(number) {
      return Number(number).toLocaleString(undefined, {minimumFractionDigits: 2, maximumFractionDigits: 2});
    },
    populate(data) {
      this.aip = {"Average Item Price": "$" + this.formatNumber(data.avgPrice)};
      this.ipo = {"Items per Order": this.formatNumber(data.itemsPerOrder)};
      this.inv = {"Inventory Value": "$0"};

      for(let i = 0; i < data.topSellingItems.length; i++) {
        let item = {};
        item.name = data.topSellingItems[i].product.itemName;
        item.quantitySold = data.topSellingItems[i].quantitySold;
        item.percentageSales = this.formatNumber(data.topSellingItems[i].percentageSales) + "%";
        item.amountSold = "$" + data.topSellingItems[i].amountSold;
        this.topItems.push(item);
      }
      /*this.topCollections = data.topSellingCollections;
      for(let i = 0; i < this.topCollections.length; i++) {
        this.topCollections[i].amountSold = "$" + this.topCollections[i].amountSold.toFixed(2).toLocaleString();
        this.topCollections[i].percentageSales = this.topCollections[i].percentageSales.toFixed(2).toString() + "%";
      }*/

    },
  },
}
</script>