<div id="app"></div>
<template>
  <v-app>
    <v-app-bar title="Fuel Calculator"
               :elevation="2"
    ></v-app-bar>
    <v-main>
      <v-form id="fuelForm" @submit.prevent="this.calculateFuel" ref="fuelForm">
        <v-container>
          <v-row>
            <v-col>
              <div class="text-h6">Race Duration</div>
            </v-col>
          </v-row>
          <v-row>
            <v-col>
              <v-text-field label="minutes"
                            v-model="this.form.raceDuration.minutes"
                            :rules="generalFieldRulesNonRequired"
              ></v-text-field>
            </v-col>
            <v-col>
              <v-text-field label="seconds"
                            v-model="this.form.raceDuration.seconds"
                            :rules="generalFieldRulesNonRequired"
              ></v-text-field>
            </v-col>
          </v-row>
          <v-row>
            <v-col>
              <div class="text-h6">Average Lap Time</div>
            </v-col>
          </v-row>
          <v-row>
            <v-col>
              <v-text-field label="minutes"
                            v-model="this.form.averageLap.minutes"
                            :rules="generalFieldRulesNonRequired"
              ></v-text-field>
            </v-col>
            <v-col>
              <v-text-field label="seconds"
                            v-model="this.form.averageLap.seconds"
                            :rules="generalFieldRulesNonRequired"
              ></v-text-field>
            </v-col>
          </v-row>
          <v-row>
            <v-col>
              <div class="text-h6">Fuel per Lap (g)</div>
              <v-text-field label="gallons"
                            v-model="this.form.fuelPerLap"
                            :rules="generalFieldRulesRequired"
              ></v-text-field>
            </v-col>
            <v-col>
              <div class="text-h6">Fuel per Lap Offset(+/-)</div>
              <v-text-field v-model="this.form.fuelOffset"
                            placeholder="0.0"
                            :rules="generalFieldRulesNonRequired"
              ></v-text-field>
            </v-col>
            <v-col>
              <div class="text-h6">Extra Laps</div>
              <v-text-field v-model="this.form.extraLaps"
                            placeholder="0"
                            :rules="generalFieldRulesNonRequired"
              ></v-text-field>
              
            </v-col>
          </v-row>
          <v-row>
            <v-col cols="4">
              <div class="text-h6">Starting Fuel (g)</div>
              <v-text-field v-model="this.form.startingFuel"
                            label="gallons"
                            :rules="generalFieldRulesNonRequired"
              ></v-text-field>
            </v-col>
            <v-col cols="2">
              <v-btn type="submit=">Calculate Fuel</v-btn>
            </v-col>
            <v-col cols="1">
              <v-btn @click="resetForm"
                     color="error">Reset</v-btn>
            </v-col>
          </v-row>
          <v-row>
            <v-col v-for="fuelStrategy in fuelStrategies" :cols="this.form.fuelOffset ? '4' : '12'">
              <v-card>
                <v-card-title>{{fuelStrategy.cardTitle}} <v-icon :icon="fuelStrategy.cardIcon"></v-icon></v-card-title>
                <v-card-text>
                  <v-container>
                    <v-row>
                      <v-col>
                        Fuel to add: <b>{{fuelStrategy.fuelToAdd}} (g)</b>
                      </v-col>
                      <v-col>
                        Fuel per Lap: <b>{{fuelStrategy.fuelConsumptionPerLap}} (g)</b>
                      </v-col>
                    </v-row>
                    <v-row>
                      <v-col>
                        Extra Laps Fuel: <b>{{fuelStrategy.extraLapsFuel}} (g)</b>
                      </v-col>
                      <v-col>
                        Total Fuel: <b>{{fuelStrategy.totalFuel}} (g)</b>
                      </v-col>
                    </v-row>
                  </v-container>
                </v-card-text>
              </v-card>
            </v-col>
          </v-row>
        </v-container>
      </v-form>
    </v-main>
  </v-app>
</template>
<script>
export default {
  data() {
    return {
      form:{
        raceDuration:{
          minutes: null,
          seconds: null
        },
        averageLap: {
          minutes: null,
          seconds: null
        },
        fuelPerLap: null,
        fuelOffset: null,
        extraLaps: null,
        startingFuel: null
      },
      fuelStrategies: [],
      generalFieldRulesRequired: [
          value => {
              if (value) return true;
              return "Required";
          },
          value => {
              if (!isNaN(parseFloat(value)) && isFinite(value)) return true;
              return "Only digits are allowed.";
          }
      ],
      generalFieldRulesNonRequired: [
          value => {
              if ((!isNaN(parseFloat(value)) && isFinite(value)) || value == null) return true;
              return "Only digits are allowed.";
          }
      ]
    }
  },
  methods:{
    //((Race Time in seconds) /(Average lap in seconds)) * Fuel Per Lap + (Extra laps * Fuel Per Lap)
    async calculateFuel(){
      this.fuelStrategies = [];
      var validation = await this.$refs.fuelForm.validate();
      if (validation.valid) {
        var fuelPerLap = parseFloat(this.form.fuelPerLap);
        if(this.form.fuelOffset){
          var fuelOffest = parseFloat(this.form.fuelOffset);
          this.createFuelStrats(fuelPerLap - fuelOffest, "Eco", "mdi-leaf");
          this.createFuelStrats(fuelPerLap, "Balanced", "mdi-scale-balance");
          this.createFuelStrats(fuelPerLap + fuelOffest, "Power", "mdi-speedometer");
        }
        else{
          this.createFuelStrats(fuelPerLap, "Balanced");
        }
        
      }
    },
    createFuelStrats(fuelPerLap, cardTitle, icon){
      var raceTimeInSeconds = ((this.form.raceDuration.minutes ? parseFloat(this.form.raceDuration.minutes) : 0) * 60) + (this.form.raceDuration.seconds ? parseFloat(this.form.raceDuration.seconds) : 0);
      var averageLapInSeconds = ((this.form.averageLap.minutes ? parseFloat(this.form.averageLap.minutes) : 0) * 60) + (this.form.averageLap.seconds ? parseFloat(this.form.averageLap.seconds) : 0);
      var initialFuel = (raceTimeInSeconds/averageLapInSeconds) * fuelPerLap;
      var extraLapsFuel = parseFloat(this.form.extraLaps) * fuelPerLap;

      var totalFuel = (initialFuel + extraLapsFuel).toFixed(2);
      this.fuelStrategies.push({
        cardTitle: cardTitle,
        cardIcon: icon,
        totalFuel: totalFuel,
        extraLapsFuel: extraLapsFuel.toFixed(2),
        fuelToAdd: (totalFuel - (this.form.startingFuel ? parseFloat(this.form.startingFuel) : 0)).toFixed(2),
        fuelConsumptionPerLap: fuelPerLap
      });
    },
    resetForm(){
      this.form.raceDuration.minutes = null;
      this.form.raceDuration.seconds = null;
      this.form.averageLap.minutes = null;
      this.form.averageLap.seconds = null;
      this.form.fuelPerLap = null;
      this.form.fuelOffset = null;
      this.form.extraLaps = null;
      this.form.startingFuel = null;
    }
  }
}
</script> 