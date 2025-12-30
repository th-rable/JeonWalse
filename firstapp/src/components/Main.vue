<template>
  <div class="grey lighten-5 pa-4">
    <v-container>
      <!-- Mode Toggle Button -->
      <v-row>
        <v-col cols="12" class="text-center pb-8 pt-2">
          <v-btn
            :outlined="!compareWolseOnly"
            :color="compareWolseOnly ? 'primary' : 'grey darken-1'"
            @click="compareWolseOnly = !compareWolseOnly"
            rounded
            large
          >
            <v-icon left>mdi-swap-horizontal-bold</v-icon>
            {{ compareButtonText }}
          </v-btn>
        </v-col>
      </v-row>

      <!-- Input Section -->
      <h2 class="text-h4 font-weight-light text-center mb-6">정보 입력</h2>
      <v-row>
        <!-- 전세 Card -->
        <v-col cols="12" md="4" v-show="!compareWolseOnly">
          <v-card outlined height="100%">
            <v-card-title>
              <v-icon left>mdi-home</v-icon>
              전세 정보
            </v-card-title>
            <v-card-text>
              <v-text-field
                label="총 보증금"
                v-model.number="jeonse.deposit"
                type="number"
                suffix="만원"
                clearable
              ></v-text-field>
              <v-text-field
                label="보증금 중 대출금"
                v-model.number="jeonse.loan"
                type="number"
                suffix="만원"
                clearable
              ></v-text-field>
              <v-text-field
                label="대출 이자율"
                v-model.number="jeonse.rate"
                type="number"
                append-icon="mdi-percent-outline"
                clearable
              ></v-text-field>
            </v-card-text>
          </v-card>
        </v-col>

        <!-- 월세 Cards -->
        <v-col v-for="(wolse, index) in wolse_options" :key="index" cols="12" :md="compareWolseOnly ? 4 : 4">
          <v-card outlined height="100%">
            <v-card-title class="d-flex justify-space-between">
              <div>
                <v-icon left class="mr-2">mdi-home-city-outline</v-icon>
                <span>월세 {{ index + 1 }}</span>
              </div>
              <v-btn icon small @click="removeWolse(index)" v-if="wolse_options.length > 1">
                <v-icon>mdi-close-circle</v-icon>
              </v-btn>
            </v-card-title>
            <v-card-text>
              <v-text-field label="총 보증금" v-model.number="wolse.deposit" type="number" suffix="만원" clearable></v-text-field>
              <v-text-field label="월세" v-model.number="wolse.rent" type="number" suffix="만원" clearable></v-text-field>
              <v-text-field label="보증금 중 대출금" v-model.number="wolse.loan" type="number" suffix="만원" clearable></v-text-field>
              <v-text-field label="대출 이자율" v-model.number="wolse.rate" type="number" append-icon="mdi-percent-outline" clearable></v-text-field>
            </v-card-text>
          </v-card>
        </v-col>
      </v-row>

      <!-- 월세 추가 버튼 (Moved to its own row) -->
      <v-row class="mt-4">
        <v-col cols="12">
          <v-card @click="addWolse" class="add-card d-flex align-center justify-center" outlined height="100%" max-height="150">
            <div class="text-center">
              <v-icon large color="primary">mdi-plus-box-outline</v-icon>
              <div class="mt-2">월세 옵션 추가</div>
            </div>
          </v-card>
        </v-col>
      </v-row>

      <!-- Settings Section -->
      <v-row class="mt-8">
        <v-col cols="12">
          <v-card outlined>
            <v-card-title class="d-flex justify-space-between align-center">
              <div>
                <v-icon left class="mr-2">mdi-tune</v-icon>
                <span>추가 설정</span>
              </div>
              <v-switch v-model="showAdvancedSettings" hide-details class="ma-0"></v-switch>
            </v-card-title>
            <v-expand-transition>
              <div v-show="showAdvancedSettings">
                <v-divider></v-divider>
                <v-card-text>
                  <v-text-field label="내 자본금" v-model.number="myCapital" type="number" suffix="만원" hint="대출 없이 사용할 수 있는 자본금이 얼마인가요?" clearable></v-text-field>
                  <v-text-field label="연간 투자 수익률" v-model.number="investmentRate" type="number" append-icon="mdi-percent-outline" hint="남은 자본금을 투자했을 때 기대되는 연간 수익률은?" clearable></v-text-field>
                </v-card-text>
              </div>
            </v-expand-transition>
          </v-card>
        </v-col>
      </v-row>
      
      <!-- Calculate Button -->
      <v-row>
        <v-col class="text-center pa-8">
          <v-btn color="primary" @click="calculate" block height="60" class="text-h5" depressed rounded>
            <v-icon left>mdi-calculator</v-icon>
            비교하기
          </v-btn>
        </v-col>
      </v-row>

      <!-- Result Section -->
      <div v-if="result.show" class="mt-4">
        <h2 class="text-h4 font-weight-light text-center mb-6">비교 결과</h2>
        <v-row>
          <v-col
            v-for="(option, index) in result.allOptions"
            :key="option.name"
            cols="12"
            md="4"
          >
            <v-card :elevation="index === 0 ? 16 : 2" :class="{ 'winner-card': index === 0 }">
              <div class="winner-badge" v-if="index === 0">
                <v-icon left>mdi-trophy</v-icon>
                가장좋음
              </div>
              <v-card-title class="justify-center text-h5 pt-8">
                {{ option.name }}
              </v-card-title>
              <v-card-text class="text-center">
                <div class="result-cost-value font-weight-bold primary--text text--darken-2">{{ formatCurrency(option.cost) }}</div>
                <div class="text-caption grey--text text--darken-1 mt-1">월 실질 비용</div>
                
                <v-divider class="my-4"></v-divider>

                <div v-if="index === 0 && result.savingAmount > 0" class="mb-4">
                  2위 대비 월 <strong class="black--text">{{ formatCurrency(result.savingAmount) }}</strong> 절약
                </div>

                <div class="text-caption text-left grey--text text--darken-2">
                  <p v-if="option.type === '월세'" class="mb-1">
                    <v-icon small left>mdi-cash</v-icon>월세: {{ formatCurrency(option.rent) }}
                  </p>
                  <p class="mb-1">
                    <v-icon small left>mdi-bank-transfer-out</v-icon>대출 이자: {{ formatCurrency(option.interest) }}
                  </p>
                  <p v-if="showAdvancedSettings" class="mb-1">
                    <v-icon small left>mdi-bank-transfer-in</v-icon>투자 수익(차감): {{ formatCurrency(option.return) }}
                  </p>
                </div>
              </v-card-text>
            </v-card>
          </v-col>
        </v-row>
      </div>
    </v-container>
  </div>
</template>

<script>
export default {
  name: 'MainComponent',
  data() {
    return {
      myCapital: 10000,
      investmentRate: 5,
      compareWolseOnly: false,
      showAdvancedSettings: false,
      jeonse: {
        deposit: 30000,
        loan: 24000,
        rate: 3.5,
      },
      wolse_options: [
        {
          deposit: 5000,
          rent: 100,
          loan: 0,
          rate: 3.5,
        }
      ],
      result: {
        show: false,
        allOptions: [],
        savingAmount: 0,
      }
    }
  },
  computed: {
    compareButtonText() {
      // Per user request, do not modify this
      return this.compareWolseOnly ? '전세를 포함해서 비교하기' : '월세만 비교하기';
    }
  },
  methods: {
    addWolse() {
      this.wolse_options.push({ deposit: 0, rent: 0, loan: 0, rate: 3.5 });
    },
    removeWolse(index) {
      if (this.wolse_options.length > 1) {
        this.wolse_options.splice(index, 1);
      }
    },
    calculate() {
      const parse = (value) => (typeof value === 'number' && value > 0 ? value : 0);
      const invRate = this.showAdvancedSettings ? parse(this.investmentRate) / 100 : 0;
      const totalCapital = this.showAdvancedSettings ? parse(this.myCapital) : 0;
      let allOptions = [];
      
      const getMonthlyReturn = (capital) => (parse(capital) * invRate) / 12;

      if (!this.compareWolseOnly) {
        const jDeposit = parse(this.jeonse.deposit);
        const jLoan = parse(this.jeonse.loan);
        const jRate = parse(this.jeonse.rate);
        
        const myMoneyForJeonse = Math.max(0, jDeposit - jLoan);
        const remainingCapitalForJeonse = totalCapital - myMoneyForJeonse;
        const jeonseMonthlyInterest = (jLoan * (jRate / 100)) / 12;
        const jeonseReturn = getMonthlyReturn(remainingCapitalForJeonse);
        const netJeonseCost = jeonseMonthlyInterest - jeonseReturn;

        allOptions.push({
          name: '전세', type: '전세', cost: netJeonseCost,
          interest: jeonseMonthlyInterest, return: jeonseReturn,
        });
      }

      this.wolse_options.forEach((wolse, index) => {
        const wDeposit = parse(wolse.deposit);
        const wLoan = parse(wolse.loan);
        const wRate = parse(wolse.rate);
        const wRent = parse(wolse.rent);

        const myMoneyForWolse = Math.max(0, wDeposit - wLoan);
        const remainingCapitalForWolse = totalCapital - myMoneyForWolse;
        const wolseMonthlyInterest = (wLoan * (wRate / 100)) / 12;
        const wolseReturn = getMonthlyReturn(remainingCapitalForWolse);
        const netWolseCost = wRent + wolseMonthlyInterest - wolseReturn;
        
        allOptions.push({
          name: `월세 ${index + 1}`, type: '월세', cost: netWolseCost,
          rent: wRent, interest: wolseMonthlyInterest, return: wolseReturn,
        });
      });

      if (allOptions.length === 0) {
        this.result.show = false;
        return;
      }

      allOptions.sort((a, b) => a.cost - b.cost);
      
      this.result.allOptions = allOptions;
      this.result.savingAmount = allOptions.length > 1 ? allOptions[1].cost - allOptions[0].cost : 0;
      this.result.show = true;
    },
    formatCurrency(value) {
      if (typeof value !== 'number') return '0원';
      return `${Math.round(value * 10000).toLocaleString('ko-KR')}원`;
    }
  }
}
</script>

<style scoped>
.add-card {
  border: 2px dashed #BDBDBD;
  cursor: pointer;
  transition: all 0.2s ease-in-out;
}
.add-card:hover {
  background-color: #F5F5F5;
  border-color: #2196F3;
}
.winner-card {
  background: linear-gradient(145deg, #FFFDE7, #FFF8E1);
  border: 1px solid #FFD700;
}
.winner-badge {
  background-color: #FFD700;
  color: black;
  font-weight: bold;
  padding: 4px 12px;
  border-radius: 0 0 16px 16px;
  display: inline-block;
  position: absolute;
  top: 0;
  left: 50%;
  transform: translateX(-50%);
}


.result-cost-value {
  font-size: 2.5rem !important; /* Custom large size */
  line-height: 1.2 !important;
}

</style>