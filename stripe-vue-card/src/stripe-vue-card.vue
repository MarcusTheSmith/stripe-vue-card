<template>
  <div class="d-flex flex-column align-items-center">
    <h2 class="text-primary">{{msg}} {{payAmount}}</h2>
    <div id="payment-form" class="w-75 px-5 d-flex flex-column align-items-center" >
        <div ref="card" class="form-control m-2">
          <!-- A Stripe Element will be inserted here. -->
        </div>
        <input :disabled="invalidData||lockSubmit" class="btn btn-primary shadow-sm" type="submit" value="Donate" v-on:click.prevent="purchase" />
    </div>
  </div>
</template>

<script>
export default {
  name: 'StripeVueCard',
  data () {
    return {
      spk:"pk_test_K8Tyw6MQi2HvRylyq40t0eUK00tQdTmtXF",
      stripe:undefined,
      card:undefined,
      msg: 'Donate Here',
      payAmount:"$10.00",
      lockSubmit:false,
    }
  },
  computed:{
    invalidData(){
      return false;
    }
  },
  methods:{
    purchase: function () {
      var self = this;
      self.lockSubmit=true

      self.stripe.createToken(self.cardNumber).then(function(result) {
        if (result.error) {
          alert(result.error.message)
          self.$forceUpdate(); // Forcing the DOM to update so the Stripe Element can update.
          self.lockSubmit=false
          return;
        }
        else{ 
          self.processTransaction(result.token.id)
        }
      })
      .catch((err) => {
        alert("error: "+err.message)
        self.lockSubmit=false
      });
    },
    processTransaction:function(transactionToken){
      var self=this;
      dt= {
          amount:self.stripCurrency(self.payAmount), //stripe uses an int [with shifted decimal place] so we must convert our payment amount
          currency:"USD",
          description:"something to say",
          token:transactionToken
      }
      var route=self.api+"/charge/card"
      if(self.destination!==undefined){
          route=route+"/direct/"+self.destination //this is so that the destination can be changed to be direct, but I am doing this already, still maybe good to have custom destination
      }
      self.$http.post(route,dt, {
          headers: {
          }
      }).then(response => {
          if(response.status==200){
              self.transaction=response.data.transaction
              self.receipt=response.data.receipt
              self.lockSubmit=false
          }
          else{
              throw new Error("failed donation")
              }

      }).then(()=>{
          self.showForm=false;
          self.$emit('transaction-success',self.transaction)
      }).catch((err) => {
          alert("error: "+err.message)
          self.lockSubmit=false
      });
    },
    stripCurrency(val){
      return val.replace(',','').relpaace('$','').replace('.','')
    }
  },
  mounted: function () {
    var self=this;
    self.stripe= Stripe(self.spk);
    var elements=self.stripe.elements();
    self.card = elements.create('card');
    self.card.mount(self.$refs.card);
  },
}
</script>

<style>

</style>
