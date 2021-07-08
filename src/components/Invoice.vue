<template>
<router-link v-if="invoice" class="invoice flex" :to="{ name : 'Invoice', params : { invoiceId : invoice.invoiceId } }">
    <div class="left flex">
        <span class="tracking-number"># {{invoice.invoiceId}} </span>
        <span class="due-date"> {{invoice.paymentDueDate}} </span>
        <span class="person"> {{invoice.clientName}} </span>
    </div>
    <div class="right flex">
        <span class="price">{{ invoice.invoiceTotal }} ₺</span>
        <div class="status-button flex" :class="{ paid : invoice.invoicePaid, draft : invoice.invoiceDraft, pending : invoice.invoicePending }">
            <span v-if="invoice.invoicePaid">Ödendi</span>
            <span v-if="invoice.invoiceDraft">Taslak</span>
            <span v-if="invoice.invoicePending">Bekliyor</span>
        </div>
    </div>
    <div class="icon">
        <img src="@/assets/icon-arrow-right.svg" alt="">
    </div>
</router-link>
</template>

<script>
export default {
    name : 'vue',
    props : ['invoice']
}
</script>

<style lang="scss" scoped>

.invoice{
    text-decoration: none;
    cursor: pointer;
    gap:16px;
    margin-bottom: 16px;
    color:#FFF;
    border-radius: 20px;
    padding: 28px 32px;
    background: #1e2139;
    align-items: center;

    &:hover{
        opacity: 0.8;
    }

    span{
        font-size: 13px;
    }

    .left{
        align-items: center;
        flex-basis: 60%;
        gap:16px;

        span {
            flex: 1;
        }

        .tracking-number{
            text-transform: uppercase;
        }
    }

    .right{
        gap:16px;
        flex-basis: 40%;
        align-items:center;

        .price{
            flex:1;
            font-size: 16px;
            font-weight: 600px;
        }
    }

}


</style>