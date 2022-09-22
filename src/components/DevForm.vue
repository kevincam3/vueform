<template>
  <v-container>
    <v-row>
      <v-col cols="12">
        <v-dialog
            v-model="dialog"
            width="500"
        >
          <v-card-text class="lighten-1">
            All fields are required.
          </v-card-text>
        </v-dialog>
        <h1 class="font-weight-bold mb-3">
          Project
        </h1>
        <v-stepper v-model="e1">
          <v-stepper-header>
            <v-stepper-step
                :complete="e1 > 1"
                step="1"
            >
              Configuration
            </v-stepper-step>

            <v-divider></v-divider>

            <v-stepper-step
                :complete="e1 > 2"
                step="2"
            >
              Summary
            </v-stepper-step>

          </v-stepper-header>

          <v-stepper-items>
            <v-stepper-content step="1">
              <v-data-table
                  :headers="headers"
                  :items="rowitems"
                  id="step1"
                  hide-default-footer
              >
                <template v-slot:item="item">
                  <tr>
                    <td>
                      <br>
                      <v-text-field
                          name="productName[]"
                          label="Product name"
                          :rules="productRules"
                          outlined
                      ></v-text-field>
                    </td>
                    <td>
                      <br>
                      <v-text-field
                          name="amount[]"
                          label="Dollar amount"
                          :rules="amountRules"
                          outlined
                      ></v-text-field>
                    </td>
                    <td>
                      <br>
                      <v-select
                          name="jobOwner[]"
                          :items="OwnerSelectOptions"
                          :rules="ownerRules"
                          item-text="name"
                          item-value="name"
                          label="Select job owner"
                          outlined
                      ></v-select>
                    </td>
                    <td class="text-center">
                      <v-icon
                          @click="deleteItem(item)"
                      >
                        mdi-delete
                      </v-icon>
                    </td>
                  </tr>
                </template>

              </v-data-table>
              <br />
              <v-btn
                  class="float-left add-row"
                  @click="addItem()"
                  text
              >
                <span>+ add row</span>
              </v-btn>
              <v-btn
                  class="float-right"
                  rounded
                  color="primary"
                  @click="next()"
              >
                Next
              </v-btn>
            </v-stepper-content>

            <v-stepper-content step="2">
              <v-col class="col col-md-6">
                <v-data-table
                    :headers="headers2"
                    :items="rowitems2"
                    hide-default-footer
                >


                </v-data-table>
              </v-col>
              <v-btn
                  class="float-left back-btn"
                  @click="e1 = 1"
                  text
              >
                <span>&#60; BACK</span>
              </v-btn>
            </v-stepper-content>

          </v-stepper-items>
        </v-stepper>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
import axios from "axios"
export default {
  name: 'DevForm',

  data () {
    return {
      e1: 1,
      dialog: false,
      OwnerSelectOptions: [],
      productName: [],
      amount: [],
      jobOwner: [],
      productRules: [
        v => !!v || 'Product Name is required',
      ],
      amountRules: [
        v => !!v || 'Dollar Amount is required',
        v => /^\$?[0-9]+(\.[0-9][0-9])?$/.test(v) || 'USD Format: 0 or 0.00',
      ],
      ownerRules: [
        v => !!v || 'Job owner is required',
      ],
      result: [],
      headers: [
        {
          text: 'Add plan name',
          sortable: false,
          name: 'plan',
          value: 'plan'
        },
        {
          text: '$ Amount',
          sortable: false,
          name: 'amount',
          value: 'amount'
        },
        {
          text: 'Job Owner',
          sortable: false,
          name: 'owner',
          value: 'owner',
        },
        { text: '', value: 'actions', sortable: false },
      ],
      headers2: [
        {
          text: '$ Sum',
          sortable: false,
          name: 'totalAmount',
          value: 'totalAmount',

        },
        {
          text: 'Job Owner',
          sortable: false,
          name: 'name',
          value: 'name',
          align: 'end',
        }
      ],
      rowitems: [{}],
      rowitems2: []
    }
  },
  async mounted() {
    let api_return = await axios.get('https://demo-api.bettercommissions.com/interview-data/users');
    this.OwnerSelectOptions = api_return.data.users;
  },
  methods: {
    deleteItem(item){
      this.rowitems.splice(item.index, 1)
    },
    addItem(){
      this.rowitems.push({})
    },
    validateField(values,type){
      for (let i = 0; i < values.length; i++) {
        if(values[i] == ''){
          return false;
        }
      }
      if(type == 'number'){
        for (let i = 0; i < values.length; i++) {
          if(!/^\$?[0-9]+(\.[0-9][0-9])?$/.test(values[i])){
            return false;
          }
        }
      }
      return true;
    },
    next(){
      this.result = []
      this.rowitems2 = []
      this.productName = []
      this.amount = []
      this.jobOwner = []

      this.productName = this.getResult('productName[]');
      this.amount = this.getResult('amount[]');
      this.jobOwner = this.getResult('jobOwner[]');

      let valid1 = true;
      let valid2 = true;
      let valid3 = true;

      valid1 = this.validateField(this.productName,'string')
      valid2 = this.validateField(this.amount,'number')
      valid3 = this.validateField(this.jobOwner,'string')

      if(valid1 && valid2 && valid3){
        let jobOwners = [];

        for (let i = 0; i < this.amount.length; i++) {
          if(!this.inArray(this.jobOwner[i],jobOwners)){
            jobOwners.push(this.jobOwner[i]);
            this.result[this.jobOwner[i]] = {
              'name': this.jobOwner[i],
              'totalAmount': parseFloat(this.amount[i])
            }
          }else{
            let currentAmount = this.result[this.jobOwner[i]].totalAmount;
            this.result[this.jobOwner[i]].totalAmount = (parseFloat(currentAmount) + parseFloat(this.amount[i]));
          }
        }


        for (const key in this.result) {
          this.rowitems2.push({totalAmount: this.result[key].totalAmount, name: this.result[key].name})
        }

        this.e1 = 2
      }else{
        this.dialog = true;
      }
    },
    getResult(selector){
      let input = document.getElementsByName(selector);

      let array = [];
      for (let i = 0; i < input.length; i++) {
        let a = input[i];
        array.push(a.value);
      }
      return array;
    },
    inArray(needle, haystack) {
      let length = haystack.length;
      for(let i = 0; i < length; i++) {
        if(haystack[i] == needle) return true;
      }
      return false;
    }
  }
}
</script>

<style>
.theme--light.v-application,
.v-stepper__items,
.theme--light.v-data-table,
.v-sheet.v-stepper {
  background-color: #F5F5F5 !important;
}
.v-sheet.v-stepper {
  box-shadow: none !important;
}
.v-stepper__header {
  margin-bottom: 10px !important;
  background-color: #FFFFFF;
}
.container {
  max-width: 1033px;
}
.v-stepper__content {
  padding: 0;
  padding-top: 24px;
}
.v-input__slot {
  background: #FFFFFF !important;
}
h1 {
  font-size: 24px;
}
.add-row span,
.back-btn span {
  color: #1976D2;
}
.theme--light.v-data-table > .v-data-table__wrapper > table > tbody > tr:hover:not(.v-data-table__expanded__content):not(.v-data-table__empty-wrapper) {
  background: transparent !important;
}
button.add-row:focus, button.add-row:hover,button.add-row,
button.back-btn:focus, button.back-btn:hover,button.back-btn {
  color: transparent !important;
}
.v-input {
  width: 234px;
  position: relative;
  top: 3px;
}
.v-input.v-select {
  width: 213px;
}
label.v-label {
  color: rgba(0, 0, 0, 0.87) !important;
}
td {
  border-bottom: 1px solid rgba(0, 0, 0, 0.12) !important;
}
.v-dialog > * {
  background: #FFFFFF;
}

@media(max-width: 767px){
  /* Force table to not be like tables anymore */
  #step1 table,
  #step1 thead,
  #step1 tbody,
  #step1 th,
  #step1 td,
  #step1 tr {
    display: block;
  }

  /* Hide table headers (but not display: none;, for accessibility) */
  #step1 thead tr {
    position: absolute;
    top: -9999px;
    left: -9999px;
  }

  #step1 tr {  }

  #step1 td {
    /* Behave  like a "row" */

    position: relative;
    height: auto !important;
  }

  #step1 td:before {
    /* Now like a table header */
    position: absolute;
    /* Top/left values mimic padding */
    top: 6px;
    left: 6px;
    width: 45%;
    padding-right: 10px;
    white-space: nowrap;
  }
  #step1 td.text-center {
    min-height: 40px;
    display: flex;
    justify-content: center;
  }
  /*
  Label the data
  */
  #step1 td:nth-of-type(1):before { content: ""; }
  #step1 td:nth-of-type(2):before { content: ""; }
  #step1 td:nth-of-type(3):before { content: ""; }
  #step1 td:nth-of-type(4):before { content: ""; }

}

@media(min-width: 768px) and (max-width: 900px) {
  .v-input {
    width: 185px;
  }
}
</style>