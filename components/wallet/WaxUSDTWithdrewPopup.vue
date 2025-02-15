<template lang="pug">
  el-dialog(:visible.sync="visible" width="500px")
    template(#title)
      .title-container
        i.el-icon-wallet
        .text Withdraw USDT from WAX
    .main(v-if="this.$store.state.user && this.$store.state.user.name")
      blockquote.blockquote.text-left
        p.text-wrap.mb-0
          | We use the EOS network as a USDT provider. When depositing, you must select the EOS network.
        footer.blockquote-footer.mt-1
          | Carefully read the instructions before making a deposit.
        footer.blockquote-footer.mt-1
          | Deposit Fee is 0.05 USDT

      img(width="100%" src="~/assets/images/wax-usdt-withdraw.png")

      h6.mt-3.mb-2.red
        | Fill in the MEMO on the exchange exactly as above
        br
        | An incorrectly filled MEMO can lead to loss of funds

      .mt-3.mb-2.w-100
        b.mt-1 CEX Deposit Address
        el-input(placeholder="Address" v-model="cex")

      .mt-2.mb-2.w-100
        b.mt-1 CEX Memo
        el-input(placeholder="MEMO" v-model="memo")

      .mt-2.mb-2.w-100
        b.mt-1 Amount
        PoolTokenInput(label="Deposit Amount" :locked="true" :token="{ decimals: 4, contract: 'usdt.alcor', symbol: 'USDT' }" v-model="amount")

      AlcorButton.w-100(outline @click="submit") {{ $t('Withdraw')}}
    .main(v-else)
      you need log in
</template>

<script>
import AlcorButton from '@/components/AlcorButton.vue'

import PoolTokenInput from '~/components/amm/PoolTokenInput'

export default {
  name: 'DepositPopup',

  components: {
    AlcorButton,
    PoolTokenInput
  },

  data: () => ({
    visible: false,
    amount: null,
    cex: null,
    memo: null
  }),

  computed: {
    memo() {
      return this.$store.state.user.name.replaceAll('.', '0')
    }
  },

  methods: {
    submit() {
      if (!this.amount) return this.$notify({ type: 'warning', message: 'Fill Amount' })
      if (parseFloat(this.amount) < 1) return this.$notify({ type: 'warning', message: 'Minimum amount 1 USDT' })

      // TODO Memo regexp
      if (!this.memo) return this.$notify({ type: 'warning', message: 'Fill CEX MEMO' })
      if (!/^[0-9]*$/.test(this.memo)) return this.$notify({ type: 'warning', message: 'Invalid CEX MEMO' })

      if (!this.cex) return this.$notify({ type: 'warning', message: 'Invalid CEX ACCOUNT' })

      const cexRegExp = new RegExp('(^[a-z1-5.]{1,11}[a-z1-5]$)|(^[a-z1-5.]{12}[a-j1-5]$)')
      if (!cexRegExp.test(this.cex)) return this.$notify({ type: 'warning', message: 'Invalid CEX ACCOUNT' })

      this.withdraw()
    },

    async withdraw() {
      const quantity = parseFloat(this.amount).toFixed(4) + ' USDT'

      const actions = [{
        account: 'usdt.alcor',
        name: 'retire',
        authorization: [this.$store.state.user.authorization],
        data: {
          owner: this.$store.state.user.name,
          quantity,
          beneficiary: 'cexdep.alcor',
          memo: this.cex + '#' + this.memo
        }
      }]

      try {
        await this.$store.dispatch('chain/sendTransaction', actions)
        this.closePopup()
        this.$notify({ type: 'success', title: 'Withdraw', message: 'Withdraw will take about 3 minutes' })
      } catch (e) {
        this.$notify({ type: 'error', title: 'Withdraw', message: 'Fill CEX MEMO' })
      }
    },

    openPopup() {
      this.visible = true
    },
    closePopup() {
      this.visible = false
    }
  }
}
</script>

<style scoped lang="scss">
.title-container {
  display: flex;
  align-items: center;

  .text {
    font-weight: 500;
    padding-left: 8px;
  }
}

.main {
  display: flex;
  flex-direction: column;
  align-items: center;
}

.qr-code {
  border-radius: 14px;
  padding: 24px;
  border-radius: 12px;
  width: 100%;
  max-width: 200px;
  background: var(--btn-active);

  img {
    object-fit: contain;
    width: 100%;
  }
}

.account-name {
  display: flex;
  align-items: center;
  color: var(--text-default);
  margin: 14px 0;
  cursor: pointer;

  .copy-container {
    margin-left: 6px;
    display: flex;
    align-items: center;
    background: var(--btn-active);
    padding: 4px 10px;
    border-radius: 4px;

    .name {
      padding-right: 6px;
      font-size: 0.86rem;
    }
  }
}

.done {
  width: 100%;
  color: var(--main-green);
  padding: 10px;
  border-radius: 10px;
}
</style>
