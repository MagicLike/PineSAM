<script setup>
import {computed} from 'vue';
import { useAppStore } from '../stores/appstore';
import { getLocalStorageValue } from '../storage';
let holdingBtnTimer = false;
let isPlusMinusHeld = false;
const store = useAppStore();
const presets = getLocalStorageValue(`presets`, ['315', '365']);
const thermometerIcon = computed(() => {
  const temp = store.liveData?.LiveTemp??0;
  if (temp < 50) return 'fa-thermometer-empty';
  if (temp < 100) return 'fa-thermometer-quarter';
  if (temp < 280) return 'fa-thermometer-half';
  if (temp < 370) return 'fa-thermometer-three-quarters';
  return 'fa-thermometer-full';
});
const thermometerHue = computed(() => {
  const tempRange = 450;
  const hueRange = 100;
  const currentTemp = store.liveData?.LiveTemp??0;
  const hue = (tempRange - currentTemp) / tempRange * hueRange;
  return hue;
});
const onPresetBtnDown = (idx) => {
  const setTemp = store.settings.SetTemperature.value;
  holdingBtnTimer = setTimeout(() => {
    const yes = window.confirm(`Are you sure you wish to set this preset to ${setTemp}?`);
    if (yes) {
      presets[idx] = setTemp;
    }
    localStorage.setItem('presets', JSON.stringify(presets));
    holdingBtnTimer = null;
  }, 1000);
};
const onPresetBtnUp = () => {
  if (holdingBtnTimer) clearTimeout(holdingBtnTimer);
  holdingBtnTimer = null;
};
const onChangeTempBtnDown = (direction) => {
  const step = store.settings.TempChangeLongStep.value;
  const updateTemperature = () => {
    isPlusMinusHeld = true;
    setExactTemperature(store.settings.SetTemperature.value + direction * step);
  };
  holdingBtnTimer = setInterval(() => {
    updateTemperature();
    clearInterval(holdingBtnTimer);
    // speed up the temperature change
    holdingBtnTimer = setInterval(updateTemperature, 600);
  }, 1000);
};
const onChangeTempBtnUp = () => {
  if (holdingBtnTimer) {
    // setTimeout to ensure "onClick" doesn't update the temperature
    setTimeout(() => {
      clearInterval(holdingBtnTimer);
      holdingBtnTimer = null;
      isPlusMinusHeld = false;
    }, 0);
  }
};
const changeTemperature = (direction) => {
  if (isPlusMinusHeld) return;
  const currentSetTemp = store.settings.SetTemperature.value;
  const step = store.settings.TempChangeShortStep.value;
  setExactTemperature(currentSetTemp + direction * step);
};
const setExactTemperature = (temp) => {
  store.setTemperature(temp);
};
</script>
<template>
  <div class="temp-holder restore-click">
    <div class="live-temp">
      <div class="buttons are-large">
        <button
          class="button change minus primary"
          @click="changeTemperature(-1)"
          @touchstart="onChangeTempBtnDown(-1)"
          @mousedown.left="onChangeTempBtnDown(-1)"
          @touchend="onChangeTempBtnUp"
          @mouseup="onChangeTempBtnUp"
        >
          <i class="fas fa-minus"></i>
        </button>
        <button
          class="button change plus primary"
          @click="changeTemperature(1)"
          @touchstart="onChangeTempBtnDown(1)"
          @mousedown.left="onChangeTempBtnDown(1)"
          @touchend="onChangeTempBtnUp"
          @mouseup="onChangeTempBtnUp"
        >
          <i class="fas fa-plus"></i>
        </button>
      </div>
      <div style="font-size: 0.3em; margin-bottom: 0.8em">
        SET {{ store.settings?.SetTemperature?.value }}&deg;{{ store.settings?.TemperatureUnit?.value ? 'F' : 'C' }}
      </div>
      <i
        class="fa"
        :class="thermometerIcon"
        :style="{ color: `hsl(${thermometerHue}, 80%, 50%)` }"
        style="font-size: 4rem;"
      />
      <span>&nbsp;</span>
      <span style="font-weight: 500">{{ store.liveData?.LiveTemp }}</span
      >&deg;{{ store.settings?.TemperatureUnit?.value ? 'F' : 'C' }}
      <div style="font-size: 0.2em; margin-top: -1em">CURRENT</div>
    </div>
    <div class="info-row">
      <div
        v-for="(presetTemp, idx) in presets"
        :key="`preset-${idx}`"
        :aria-valuenow="presetTemp"
        class="row-item preset button"
        @click="setExactTemperature(presetTemp)"
        @mousedown.left="onPresetBtnDown(idx)"
        @mouseup="onPresetBtnUp"
        @touchstart="onPresetBtnDown(idx)"
        @touchend="onPresetBtnUp"
      >
        <div class="icon-text has-text-info">
          <span class="icon is-large primary">
            <i class="fas fa-thermometer-half"></i>
            <span class="preset">{{ presetTemp }}</span>
          </span>
        </div>
      </div>
      <div class="row-item power">
        <div class="icon-text">
          <span class="icon has-text-dark" style="width: 100%">
            <i class="fa fa-plug"></i>
            <span>{{ store.liveData?.Voltage }} V</span>
          </span>
        </div>
      </div>
      <div class="row-item power">
        <div class="icon-text">
          <span class="icon has-text-dark" style="width: 100%">
            <i class="fa fa-bolt"></i>
            <span>{{ store.liveData?.Watts }} W</span>
          </span>
        </div>
      </div>
      <div class="row-item power">
        <div class="icon-text">
          <span class="icon has-text-dark" style="width: 100%">
            <i class="fas fa-angle-double-up"></i>
            <span>{{ store.liveData?.PeakWatts }} W</span>
          </span>
        </div>
      </div>
    </div>
  </div>
</template>
<style scoped>
.temp-holder {
  display: flex;
  flex-flow: column;
  align-items: center;
  width: 350px;
}
.live-temp {
  border: 1px solid #c0cbcd;
  font-size: 5em;
  width: 320px;
  height: 230px;
  position: relative;
  border-radius: 5px;
  border-bottom-left-radius: 0;
  border-bottom-right-radius: 0;
  text-align: center;
}
.live-temp button.change {
  position: absolute;
  top: 0;
  margin: -1px;
}
.live-temp .change:first-child {
  border-top-right-radius: 0px;
  border-bottom-left-radius: 0px;
}
.live-temp .change:last-child {
  border-top-left-radius: 0px;
  border-bottom-right-radius: 0px;
}
.change i {
  font-size: 1.9rem;
}
.live-temp .minus {
  left: 0;
}
.live-temp .plus {
  right: 0;
}
.info-row {
  display: flex;
  flex-wrap: wrap;
  justify-content: stretch;
  align-content: stretch;
  height: 100px;
  width: 320px;
}
div.preset {
  border-radius: 0;
  border-top:0;
  border-bottom:0;
  width: 50%;
  height: 75px;
  font-weight: 500;
  background-color: rgba(var(--secondary-color),0.1);
}
div.preset:first-child {
  border-left-width: 1px;
}
div.row-item {
  border-left-width:0px;
}
.power:last-child {
  border-bottom-right-radius: 5px;
}
.power:nth-child(3) {
  border-bottom-left-radius: 5px;
  border-left-width:1px;
}
span.preset {
  font-size: 1.7em;
}
.row-item {
  min-width: 33.33%;
  height: 50%;
  font-size: 1.2em;
  border: 1px solid #c0cbcd;
  display: flex;
  align-items: center;
  justify-content: center;
  text-align: center;
}
.row-item i {
  font-size: 1.2em;
  font-weight: 600;
  margin-right: 0.4em;
}
</style>
