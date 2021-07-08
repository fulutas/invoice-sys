<template>
  <div class="invoice-wrap flex flex-column" @click="checkClick" ref="invoiceWrap">
      <form @submit.prevent="submitForm" class="invoice-content">
        <Loading v-show="loading"/>
        <h1 v-if="!editInvoice">Yeni Fatura</h1>
        <h1 v-else>Fatura Düzenle</h1>

        <!-- Bill Form -->
        <div class="bill-from flex flex-column">
            <h4>Fatura Düzenleyen</h4>
            <div class="input flex flex-column">
                <label for="billerStreetAddress">Adres</label>
                <input type="text" id="billerStreetAddress" v-model="billerStreetAddress"  required>
            </div>
            <div class="location-details flex">
                <div class="input flex flex-column">
                    <label for="billerCity">Şehir</label>
                    <input type="text" id="billerCity" v-model="billerCity"  required>
                </div>
                <div class="input flex flex-column">
                    <label for="billerZipCode">Posta Kodu</label>
                    <input type="text" id="billerZipCode" v-model="billerZipCode"  required>
                </div>              
                <div class="input flex flex-column">
                    <label for="billerCountry">Ülke</label>
                    <input type="text" id="billerCountry" v-model="billerCountry"  required>
                </div>                                                              
            </div>
        </div>

        <!-- Bill To -->
        <div class="bill-to flex flex-column">
            <h4>Müşteri</h4>
            <div class="input flex flex-column">
                <label for="clientName">Müşteri Adı</label>
                <input type="text" id="clientName" v-model="clientName"  required>
            </div>
            <div class="input flex flex-column">
                <label for="clientEmail">Müşteri Email</label>
                <input type="text" id="clientEmail" v-model="clientEmail"  required>
            </div>
            <div class="input flex flex-column">
                <label for="clientStreetAddress">Adres</label>
                <input type="text" id="clientStreetAddress" v-model="clientStreetAddress"  required>
            </div>

            <div class="location-details flex">
                <div class="input flex flex-column">
                    <label for="clientCity">Şehir</label>
                    <input type="text" id="clientCity" v-model="clientCity"  required>
                </div>
                <div class="input flex flex-column">
                    <label for="clientZipCode">Posta Kodu</label>
                    <input type="text" id="clientZipCode" v-model="clientZipCode"  required>
                </div>              
                <div class="input flex flex-column">
                    <label for="clientCountry">Ülke</label>
                    <input type="text" id="clientCountry" v-model="clientCountry"  required>
                </div>                                                              
            </div>
            
        </div>

        <!-- Invoice Work Details -->
        <div class="invoice-work flex flex-column">
            <div class="payment flex">
                <div class="input flex flex-column">
                    <label for="invoiceDate">Fatura Tarihi</label>
                    <input disabled type="text" id="invoiceDate" v-model="invoiceDate" required>
                </div>
                <div class="input flex flex-column">
                    <label for="paymentDueDate">Ödeme Vade Tarihi</label>
                    <input disabled type="text" id="paymentDueDate" v-model="paymentDueDate" required>
                </div>
            </div>
            <div class="input flex flex-column">
                <label for="paymentTerms">Ödeme Vade Süresi</label>
                <select type="text" id="paymentTerms" v-model="paymentTerms" required>
                    <option value="15">Net 15 Days</option>
                    <option value="30">Net 30 Days</option>
                    <option value="60">Net 60 Days</option>
                </select>
            </div>
            <div class="input flex flex-column">
                <label for="productDescription">Fatura Açıklama</label>
                <input type="text" id="productDescription" v-model="productDescription" required>
            </div>     
          <div class="work-items">
            <h3>Fatura Kalemleri</h3>
            <table class="item-list">
                <tr class="table-heading flex">
                    <th class="item-name">Madde Adı</th>
                    <th class="qty">Miktar</th>
                    <th class="price">Fiyat</th>
                    <th class="total">Toplam Tutar</th>
                </tr>
                <tr class="table-items flex" v-for="(item,index) in invoiceItemList" :key="index">
                    <td class="item-name"><input type="text" v-model="item.itemName"></td>
                    <td class="qty"><input type="text" v-model="item.qty"></td>
                    <td class="price"><input type="text" v-model="item.price"></td>
                    <td class="total flex">₺ {{ item.total = item.qty * item.price }}</td>
                    <img @click="deleteInvoiceItem(item.id)" src="@/assets/icon-delete.svg" alt="">
                </tr>
            </table>
            <div class="not-found-item" v-if="!invoiceItemList.length">
                <p>Henüz bir kayıt eklemediniz.</p>
            </div>
            <div @click="addNewInvoiceItem" class="flex button">
                <img src="@/assets/icon-plus.svg" alt="">
                Satır Ekle
            </div>
        </div>       
        </div>

        <!-- Save/Exit Actions Button -->
        <div class="save flex">
            <div class="left">
                <button type="button" class="red" @click="closeInvoice">İptal</button>
            </div>
            <div class="right flex">
                <button v-if="!editInvoice" type="submit" class="dark-purple" @click="saveDraft">Taslak Kaydet</button>
                <button v-if="!editInvoice" type="submit" class="purple" @click="publishInvoice">Faturayı Oluştur</button>
                <button v-if="editInvoice"  type="submit" class="purple">Değişiklikleri Kaydet</button>
            </div>
        </div>
      </form>
  </div>
</template>

<script>
import db from "../firebase/firebaseInit"
import Loading from "../components/Loading.vue"

import {mapMutations, mapState, mapActions } from "vuex";
import {uid} from "uid"

export default {
 name : 'InvoiceModal',
 components : {
     Loading
 },
 data(){
    return {
      dateOptions : {year : "numeric", month : "long", day : "numeric"},
      docId: null,
      loading: null,
      billerStreetAddress: null,
      billerCity: null,
      billerZipCode: null,
      billerCountry: null,
      clientName: null,
      clientEmail: null,
      clientStreetAddress: null,
      clientCity: null,
      clientZipCode: null,
      clientCountry: null,
      invoiceDateUnix: null,
      invoiceDate: null,
      paymentTerms: null,
      paymentDueDateUnix: null,
      paymentDueDate: null,
      productDescription: null,
      invoicePending: null,
      invoiceDraft: null,
      invoiceItemList: [],
      invoiceTotal: 0,
    }
  },
  created(){
      // yeni fatura oluştururken tarih ekler.
      if(!this.editInvoice){
        this.invoiceDateUnix = Date.now();
        this.invoiceDate = new Date(this.invoiceDateUnix).toLocaleString('tr-TR', this.dateOptions)
      }

      // düzenle yapıldı ise;  
      if(this.editInvoice){
      const currentInvoice = this.currentInvoiceArray[0]; // state'den alır

      this.docId = currentInvoice.docId;
      this.billerStreetAddress = currentInvoice.billerStreetAddress;
      this.billerCity = currentInvoice.billerCity;
      this.billerZipCode = currentInvoice.billerZipCode;
      this.billerCountry = currentInvoice.billerCountry;
      this.clientName = currentInvoice.clientName;
      this.clientEmail = currentInvoice.clientEmail;
      this.clientStreetAddress = currentInvoice.clientStreetAddress;
      this.clientCity = currentInvoice.clientCity;
      this.clientZipCode = currentInvoice.clientZipCode;
      this.clientCountry = currentInvoice.clientCountry;
      this.invoiceDateUnix = currentInvoice.invoiceDateUnix;
      this.invoiceDate = currentInvoice.invoiceDate;
      this.paymentTerms = currentInvoice.paymentTerms;
      this.paymentDueDateUnix = currentInvoice.paymentDueDateUnix;
      this.paymentDueDate = currentInvoice.paymentDueDate;
      this.productDescription = currentInvoice.productDescription;
      this.invoicePending = currentInvoice.invoicePending;
      this.invoiceDraft = currentInvoice.invoiceDraft;
      this.invoiceItemList = currentInvoice.invoiceItemList;
      this.invoiceTotal = currentInvoice.invoiceTotal;
      }
  },
  methods: {

    ...mapMutations(['TOGGLE_INVOICE','TOGGLE_MODAL','TOGGLE_EDIT_INVOICE']),
    ...mapActions(['UPDATE_INVOICE','GET_INVOICES']),

    // Invoice modal kapsayıcı dışına tıklandığında popup..
    checkClick(e){
        if(e.target === this.$refs.invoiceWrap){
            this.TOGGLE_MODAL(); 
        }
    },

    closeInvoice(){
        this.TOGGLE_INVOICE()
        if(this.editInvoice){
            this.TOGGLE_EDIT_INVOICE()
        }
    },

    addNewInvoiceItem(){
        this.invoiceItemList.push({
            id : uid(),
            itemName : "",
            qty : "",
            price : 0,
            total : 0,
        })
    },

    deleteInvoiceItem(id){
        this.invoiceItemList = this.invoiceItemList.filter(item => item.id !== id)
    },

    calInvoiceTotal(){
        this.invoiceTotal = 0;
        this.invoiceItemList.forEach(item => {
            this.invoiceTotal += item.total
        })
    },

    publishInvoice(){
        this.invoicePending = true;
    },

    saveDraft(){
        this.invoiceDraft = true;
    },

    async uploadInvoice(){
        if(this.invoiceItemList.length <= 0){
            alert("Fatura kalemlerini giriniz.")
            return;
        } else {

        this.loading = true;    

        this.calInvoiceTotal(); // fatura kalemi toplamını hesapla.

        const dataBase = db.collection('invoices').doc();

        await dataBase.set({
            invoiceId: uid(6), // default 11 karakterdir.
            billerStreetAddress: this.billerStreetAddress,
            billerCity: this.billerCity,
            billerZipCode: this.billerZipCode,
            billerCountry: this.billerCountry,
            clientName: this.clientName,
            clientEmail: this.clientEmail,
            clientStreetAddress: this.clientStreetAddress,
            clientCity: this.clientCity,
            clientZipCode: this.clientZipCode,
            clientCountry: this.clientCountry,
            invoiceDate: this.invoiceDate,
            invoiceDateUnix: this.invoiceDateUnix,
            paymentTerms: this.paymentTerms,
            paymentDueDate: this.paymentDueDate,
            paymentDueDateUnix: this.paymentDueDateUnix,
            productDescription: this.productDescription,
            invoiceItemList: this.invoiceItemList,
            invoiceTotal: this.invoiceTotal, // fatura kalemin toplamı (methods da hesaplandı)
            invoicePending: this.invoicePending,
            invoiceDraft: this.invoiceDraft,
            invoicePaid: null,
            });
            
            this.loading = false;    
            this.TOGGLE_INVOICE();
            this.GET_INVOICES();
        }



    },

    async updateInvoice(){
        if(this.invoiceItemList.length <= 0){
            alert("Fatura kalemlerini giriniz.")
            return;
        } else {

        this.loading = true;    

        this.calInvoiceTotal(); // fatura kalemi toplamını hesapla.

        const dataBase = db.collection('invoices').doc(this.docId);

        
        await dataBase.update({
            billerStreetAddress: this.billerStreetAddress,
            billerCity: this.billerCity,
            billerZipCode: this.billerZipCode,
            billerCountry: this.billerCountry,
            clientName: this.clientName,
            clientEmail: this.clientEmail,
            clientStreetAddress: this.clientStreetAddress,
            clientCity: this.clientCity,
            clientZipCode: this.clientZipCode,
            clientCountry: this.clientCountry,
            paymentTerms: this.paymentTerms,
            paymentDueDate: this.paymentDueDate,
            paymentDueDateUnix: this.paymentDueDateUnix,
            productDescription: this.productDescription,
            invoiceItemList: this.invoiceItemList,
            invoiceTotal: this.invoiceTotal,
        });
            
            this.loading = false;

            const data = {
                docId : this.docId,
                routeId : this.$route.params.invoiceId
            }

            this.UPDATE_INVOICE(data);
        }



    },

    submitForm(){
        if(this.editInvoice){
            this.updateInvoice()
            return;
        } else {
            this.uploadInvoice();
        }
    }
  },

  computed: {
      ...mapState(['editInvoice', 'currentInvoiceArray'])
  },

  watch: {
      paymentTerms(){
        const futureDate = new Date(); // Mon Jul 05 2021 12:24:27 GMT+0300 (GMT+03:00)
        this.paymentDueDateUnix = futureDate.setDate(futureDate.getDate() + parseInt(this.paymentTerms));
        this.paymentDueDate = new Date(this.paymentDueDateUnix).toLocaleDateString('tr-TR', this.dateOptions)
      }
  }
}
</script>

<style lang="scss" scoped>
.invoice-wrap{
    position: fixed;
    top:0;
    left:0;
    background-color: transparent;
    width: 100%;
    height: 100vh;
    overflow: scroll;
    &::-webkit-scrollbar {
        display: none;
    }
    @media(min-width:900px){
        left: 130px;
    }

    .invoice-content{
        position: relative;
        padding: 56px;
        max-width: 700px;
        width: 100%;
        background-color: #141625;
        color: #FFF;
        box-shadow: 10px 4px 6px -1px rgba(0, 0, 0, 0.2), 0 2px 4px -1px rgba(0, 0, 0, 0.06);

        h1{
            margin-bottom: 48px;
            color:#FFF;
        }

        h3{
            margin-bottom: 16px;
            font-size: 18px;
            color:#777f98;
        }

        h4 {
            color : #7c5dfa;
            font-size: 12px;
            margin-bottom: 24px;
        }

        // Bill To / Bill From
        .bill-to,
        .bill-from{
            margin-bottom: 48px;

            .location-details {
                gap: 16px;
                div{
                    flex: 1;
                }
            }
        }

        // Invoice Work

        .invoice-work{
            .payment{
                gap: 24px;
                div{
                    flex: 1;
                }
            }

            .work-items {
                .item-list{
                    width: 100%;

                    // Item Table Styling
                    .table-heading,
                    .table-items{
                        gap : 16px;
                        font-size: 12px;

                        .item-name{
                            flex-basis: 50%;
                        }

                        .qty{
                            flex-basis: 10%;
                        }

                        .price{
                            flex-basis: 20%;
                        }

                        .total{
                            flex-basis: 20%;
                            align-self: center;
                        }
                    }

                    .table-heading{
                        margin-bottom: 16px;

                        th{
                            text-align: left;
                        }
                    }

                    .table-items{
                        position: relative;
                        margin-bottom: 24px;

                        img{
                            position: absolute;
                            top: 15px;
                            right: 0;
                            width: 12px;
                            height: 16px;
                        }
                    }
                }

                .button{
                    color:#FFF;
                    background-color: #252945;
                    align-items: center;
                    justify-content: center;
                    width: 100%;

                    img{
                        margin-right: 4px;
                    }
                }

                .not-found-item{
                    p {
                        text-align: center;
                        margin-bottom: 16px;
                        color:#777f98;
                    }
                }
            }
        }
        .save{
           margin-top: 60px;
           div{
               flex: 1;
           }
           .right{
               justify-content: flex-end;
           }
        }        
    }

    .input{
        margin-bottom: 24px;
    }

    label {
        font-size: 12px;
        margin-bottom: 6px;
    }

    input,
    select {
        width: 100%;
        background-color: #1e2139;
        color:#FFF;
        border-radius: 4px;
        padding: 12px 4px;
        border: none;

        &:focus{
            outline: none;
        }
    }
}
</style>