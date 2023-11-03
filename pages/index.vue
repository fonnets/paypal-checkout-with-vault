<template>
  <section class="pt-4 px-4">
    <div id="paypal-button" />
  </section>
</template>

<script>
export default {
  name: 'IndexPage',
  layout: 'minimal',
  data() {
    return {
      btToken: 'sandbox_xh8vxq5k_p5xmdbvs52ph5w4b',
      clientInstance: null,
      error: null,
    }
  },
  async beforeMount() {
    await this.setBtClient()
  },
  methods: {
    async setBtClient() {
      const client = await import('braintree-web/client')

      const that = this
      client
          .create({
            authorization: this.btToken,
          })
          .then(function (clientInstance) {
            that.clientInstance = clientInstance
            that.setPaypalCheckoutWithVault()
          })
          .catch(function (err) {
            that.error = err
          })
    },
    async setPaypalCheckoutWithVault() {
      document.querySelector('#paypal-button').innerHTML = ''
      const payPalCheckout = await import('braintree-web/paypal-checkout')
      try {
        const paypalCheckoutInstance = await payPalCheckout.create({
          client: this.clientInstance,
        })

        if (paypalCheckoutInstance) {
          paypalCheckoutInstance.loadPayPalSDK(
              {
                currency: 'EUR',
                components: 'buttons,messages',
                'enable-funding': 'paylater',
                dataAttributes: {
                  amount: '100.00'
                },
                // autoSetDataUserIdToken: true
              },
              function () {
                const button = window.paypal.Buttons({
                  style: {
                    color: 'blue',
                  },
                  createOrder() {
                    return paypalCheckoutInstance.createPayment({
                      flow: 'checkout',
                      amount: '100.00',
                      currency: 'EUR',
                      requestBillingAgreement: true,
                      autoSetDataUserIdToken: true
                    })
                  },
                  onApprove(data, actions) {
                    // eslint-disable-next-line node/handle-callback-err
                    return paypalCheckoutInstance.tokenizePayment(
                        data,
                        // eslint-disable-next-line node/handle-callback-err
                        function (err, payload) {
                          // Submit 'payload.nonce' to your server
                        }
                    )
                  },
                })

                if (!button.isEligible()) {
                  // Skip rendering if not eligible
                  return
                }

                button.render('#paypal-button')
              }
          )
        }
      } catch (e) {
        this.error = e
      }
    },
  },
}
</script>
