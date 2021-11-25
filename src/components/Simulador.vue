<template>
  <v-container fill-height>
    <v-row justify="center" align="center" align-content="center">
      <v-card tile max-width="760px" max-height="610px">
        <v-card-title>
          Simulador de Investimento em Fotovoltaica
        </v-card-title>
        <v-card-text>
          <v-stepper v-model="currentStep" vertical tile color="info">
            <v-stepper-step :complete="monthlyBill > 100 && energyCost > 0" step="1">
              <h3>
                Consumo estimado: <span class="title">{{ (monthlyBill / energyCost).toLocaleString("pt-BR", gwh) }} KWh/Mês</span>
              </h3>
            </v-stepper-step>
            <v-stepper-content step="1">
              <v-card tile color="grey lighten-4" class="pt-3">
                <v-card-text class="pt-4">
                  <v-row class="pt-4">
                    <v-col cols="12" order="0" order-sm="2" sm="6">
                      <v-slider dense hide-details v-model="monthlyBill" min="100" max="100000" step="1"> </v-slider>
                    </v-col>
                    <v-col cols="12" order="2" order-sm="3" sm="6">
                      <v-slider dense hide-details v-model="energyCost" min="0.01" max="3" step="0.01"></v-slider>
                    </v-col>
                    <v-col cols="12" order="1" order-sm="0" sm="6">
                      <v-text-field readonly hide-details type="number" label="Custo mensal em R$" v-model.number="monthlyBill" prefix="R$"> </v-text-field>
                    </v-col>
                    <v-col cols="12" order="3" order-sm="1" sm="6">
                      <v-text-field readonly hide-details type="number" label="Valor do kW/h" v-model.number="energyCost" suffix="R$/KW/h"></v-text-field>
                    </v-col>
                  </v-row>
                </v-card-text>
                <v-card-actions>
                  <v-btn tile block @click="currentStep = 2" color="primary">Confirmar</v-btn>
                </v-card-actions>
              </v-card>
            </v-stepper-content>
            <v-stepper-step :complete="city ? true : false" step="2">
              <h3>
                {{ city ? "Localização: " : "Defina o local de instalação" }}
                <span v-if="city" class="title">{{ `${state.text} - ${city.text} ~ ${(city.value / 1000).toLocaleString("pt-BR", kwh)} KWh/m²` }}</span>
              </h3>
            </v-stepper-step>
            <v-stepper-content step="2">
              <v-card>
                <v-card-text>
                  <v-row>
                    <v-col cols="12">
                      <v-switch label="Paineis instalados em telhado?" v-model="isRoof"></v-switch>
                    </v-col>
                    <v-col cols="12" sm="6">
                      <v-select :items="states" return-object v-model="state" label="Estado:"></v-select>
                    </v-col>
                    <v-col cols="12" sm="6">
                      <v-select :items="state.value" v-model="city" return-object label="Município:"></v-select>
                    </v-col>
                  </v-row>
                </v-card-text>
                <v-card-actions>
                  <v-btn tile block :disabled="!city" @click="calculateEverything()" color="primary">Confirmar</v-btn>
                </v-card-actions>
              </v-card>
            </v-stepper-content>
            <v-stepper-step step="3" :complete="solarRadiationLevel > 0">
              <h3>Resultado:</h3>
            </v-stepper-step>
            <v-stepper-content step="3">
              <v-card>
                <v-card-text>
                  <v-row>
                    <v-col cols="12">
                      <div>
                        <h3>Potência pico do sistema: {{ peakPotency ? peakPotency.toLocaleString("pt-BR", kwh) : "---" }} KWp</h3>
                      </div>
                      <div>
                        <h3>Área total necessária: {{ totalArea ? totalArea.toLocaleString("pt-BR", area) : "---" }}m²</h3>
                      </div>
                      <div>
                        <h3>Energia gerada por ano: {{ yearGen ? yearGen.toLocaleString("pt-BR", kwh) : "---" }} KWh/Ano</h3>
                      </div>
                      <div class="d-flex">
                        <h3>Energia média gerada por mês: {{ monthGen ? monthGen.toLocaleString("pt-BR", kwh) : "---" }} KWh/Mês</h3>
                        <v-spacer></v-spacer>
                        <v-btn tile color="primary">Solicitar oraçamento</v-btn>
                      </div>
                    </v-col>
                  </v-row>
                </v-card-text>
                <v-card-actions>
                  <v-btn @click="resetForm()" color="primary" tile block>Reiniciar</v-btn>
                </v-card-actions>
              </v-card>
            </v-stepper-content>
          </v-stepper>
        </v-card-text>
      </v-card>
    </v-row>
  </v-container>
</template>

<script>
export default {
  data() {
    return {
      area: {
        style: "decimal",
        maximumFractionDigits: 2,
      },
      kwh: {
        style: "decimal",
        maximumFractionDigits: 2,
        minimumFractionDigits: 1,
      },
      gwh: {
        style: "decimal",
        maximumFractionDigits: 0,
        minimumFractionDigits: 0,
      },
      R$: {
        style: "currency",
        currency: "BRL",
        maximumFractionDigits: 2,
        minimumFractionDigits: 2,
      },
      currentStep: 1,
      step: [false, false, true, true],
      isRoof: false,
      monthlyBill: 5000,
      energyCost: 0.75,
      peakPotency: 0,
      averageConsume: 0,
      performanceRatio: 0.83,
      solarRadiationLevel: 0,
      totalArea: 0,
      pMod: 0.45,
      aMod: 2.21,
      f: 2.3,
      yearGen: 0,
      monthGen: 0,
      state: [],
      city: false,
    };
  },
  methods: {
    resetForm() {
      this.currentStep = 1;
      this.step = [false, false, true, true];
      this.isRoof = false;
      this.monthlyBill = 5000;
      this.energyCost = 0.75;
      this.peakPotency = 0;
      this.averageConsume = 0;
      this.performanceRatio = 0.83;
      this.solarRadiationLevel = 0;
      this.totalArea = 0;
      this.pMod = 0.45;
      this.aMod = 2.21;
      this.f = 2.3;
      this.yearGen = 0;
      this.monthGen = 0;
      this.state = [];
      this.city = false;
    },
    calculateEverything() {
      this.isRoof ? (this.f = 2.3): (this.f = 1.5)
      this.solarRadiationLevel = this.city.value;

      this.calculateAverageConsume();
      this.calculatePeakPotency();
      this.calculateTotalArea();
      this.calculateYearGeneration();
      this.calculateMonthGeneration();
      this.currentStep = 3;
    },
    calculateAverageConsume() {
      this.averageConsume = this.monthlyBill / this.energyCost;
    },
    calculatePeakPotency() {
      this.peakPotency = this.averageConsume / (this.performanceRatio * (this.solarRadiationLevel / 1000) * (365 / 12));
    },
    calculateTotalArea() {
      this.totalArea = (this.peakPotency / this.pMod) * this.aMod * this.f;
    },
    calculateYearGeneration() {
      this.yearGen = this.peakPotency * this.performanceRatio * (this.solarRadiationLevel / 1000) * 365;
    },
    calculateMonthGeneration() {
      this.monthGen = this.yearGen / 12;
    },
  },
  computed: {
    states: {
      get() {
        return this.$store.getters.getStates;
      },
    },
  },
};
</script>

<style></style>
