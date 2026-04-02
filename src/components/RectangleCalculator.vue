<script setup>
import { ref, computed, watch } from 'vue'
import InputNumber from 'primevue/inputnumber'
import Slider from 'primevue/slider'
import FloatLabel from 'primevue/floatlabel'

const inputArea = ref(100)
const area = ref(100)
const isStarted = ref(false)

const sideA = ref(10)
const sideB = ref(10)

const startCalculator = () => {
    if (inputArea.value && inputArea.value > 0) {
        area.value = inputArea.value
        let sqrt = Math.sqrt(area.value)
        sideA.value = Number(sqrt.toFixed(2))
        sideB.value = Number(sqrt.toFixed(2))
        isStarted.value = true
    }
}

const updateSideA = (val) => {
    if (val === null || val === undefined) return
    let newVal = Math.max(1, Math.min(val, area.value))
    sideA.value = Number(newVal.toFixed(2))
    sideB.value = Number((area.value / sideA.value).toFixed(2))
}

const updateSideB = (val) => {
    if (val === null || val === undefined) return
    let newVal = Math.max(1, Math.min(val, area.value))
    sideB.value = Number(newVal.toFixed(2))
    sideA.value = Number((area.value / sideB.value).toFixed(2))
}

const svgSize = 100
const maxDisplaySize = 90

const rectVisual = computed(() => {
    const sA = sideA.value || 1
    const sB = sideB.value || 1
    const maxSide = Math.max(sA, sB, 1)
    const scaleFactor = maxDisplaySize / maxSide
    
    const width = sA * scaleFactor
    const height = sB * scaleFactor
    
    return {
        width,
        height,
        x: (svgSize - width) / 2,
        y: (svgSize - height) / 2
    }
})

// Max limits for sliders based on the constraint that min side is 1
const maxSliderLimit = computed(() => Math.max(area.value, 1))

</script>

<template>
  <div class="bg-slate-800/50 backdrop-blur-md rounded-3xl p-8 shadow-2xl border border-slate-700 w-full">
    
    <!-- Initial Setup -->
    <div v-if="!isStarted" class="flex flex-col items-center justify-center space-y-6 py-12 animate-fade-in">
        <h2 class="text-2xl font-bold text-white">Let's start calculating!</h2>
        <FloatLabel>
            <InputNumber id="areaInput" v-model="inputArea" mode="decimal" :min="1" class="w-64" :minFractionDigits="0" :maxFractionDigits="2" autofocus @keyup.enter="startCalculator"/>
            <label for="areaInput">Enter Total Area</label>
        </FloatLabel>
        <button 
            @click="startCalculator" 
            class="px-8 py-3 rounded-full bg-teal-500 hover:bg-teal-400 text-white font-semibold transition-all shadow-lg hover:shadow-teal-500/30"
        >
            Start
        </button>
    </div>

    <!-- Active Calculator -->
    <div v-else class="grid grid-cols-1 lg:grid-cols-2 gap-12 animate-fade-in relative">
        <button 
            @click="isStarted = false" 
            class="absolute -top-4 -right-4 text-slate-400 hover:text-white transition-colors"
            title="Reset Area"
        >
            <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6" fill="none" viewBox="0 0 24 24" stroke="currentColor">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="1" d="M6 18L18 6M6 6l12 12" />
            </svg>
        </button>

        <!-- Controls -->
        <div class="flex flex-col space-y-10 justify-center">
            <div class="flex items-center justify-between text-xl font-medium">
                <span class="text-slate-300">Total Area</span>
                <span class="text-teal-400 font-bold bg-teal-900/30 px-4 py-1 rounded-full">{{ area }}</span>
            </div>

            <!-- Side A Control -->
            <div class="space-y-4 p-6 bg-slate-900/50 rounded-2xl border border-slate-700/50">
                <div class="flex justify-between items-center mb-2">
                    <label class="text-slate-300 font-semibold sm:w-[400px]">Side A (Width)</label>
                    <InputNumber 
                        :modelValue="sideA" 
                        @update:modelValue="updateSideA" 
                        mode="decimal" 
                        showButtons 
                        :min="1" 
                        :max="maxSliderLimit"
                        :minFractionDigits="0" 
                        :maxFractionDigits="2"
                        inputClass="text-right"
                        fluid
                    />
                </div>
                <Slider 
                    :modelValue="sideA" 
                    @update:modelValue="updateSideA" 
                    :min="1" 
                    :max="maxSliderLimit" 
                    :step="0.1" 
                    class="w-full"
                />
            </div>

            <!-- Side B Control -->
            <div class="space-y-4 p-6 bg-slate-900/50 rounded-2xl border border-slate-700/50">
                <div class="flex justify-between items-center mb-2">
                    <label class="text-slate-300 font-semibold sm:w-[400px]">Side B (Height)</label>
                    <InputNumber 
                        :modelValue="sideB" 
                        @update:modelValue="updateSideB" 
                        mode="decimal" 
                        showButtons
                        :min="1" 
                        :max="maxSliderLimit"
                        :minFractionDigits="0" 
                        :maxFractionDigits="2"
                        inputClass="text-right"
                        fluid
                    />
                </div>
                <Slider 
                    :modelValue="sideB" 
                    @update:modelValue="updateSideB" 
                    :min="1" 
                    :max="maxSliderLimit" 
                    :step="0.1" 
                    class="w-full"
                />
            </div>
        </div>

        <!-- Visualization -->
        <div class="flex flex-col items-center justify-center p-8 bg-slate-900/80 rounded-2xl border border-slate-700/50 relative overflow-hidden group">
            <h3 class="absolute top-4 left-6 text-slate-400 font-semibold uppercase tracking-wider text-sm">Live Visualization</h3>
            <svg 
                :viewBox="`0 0 ${svgSize} ${svgSize}`" 
                class="w-full h-full max-w-[300px] max-h-[300px] drop-shadow-2xl transition-all duration-300 ease-out"
            >
                <!-- Grid background for scale context -->
                <defs>
                    <pattern id="grid" width="10" height="10" patternUnits="userSpaceOnUse">
                        <path d="M 10 0 L 0 0 0 10" fill="none" stroke="rgba(255,255,255,0.05)" stroke-width="0.5"/>
                    </pattern>
                </defs>
                <rect width="100" height="100" fill="url(#grid)" rx="4"/>
                
                <!-- The dynamic rectangle -->
                <rect
                    :x="rectVisual.x"
                    :y="rectVisual.y"
                    :width="rectVisual.width"
                    :height="rectVisual.height"
                    rx="0"
                    fill="url(#rectGradient)"
                    stroke="#14b8a6"
                    stroke-width="0.5"
                    class="transition-all duration-300 ease-out"
                />
                
                <defs>
                    <linearGradient id="rectGradient" x1="0%" y1="0%" x2="100%" y2="100%">
                        <stop offset="0%" stop-color="#2dd4bf" stop-opacity="0.6" />
                        <stop offset="100%" stop-color="#3b82f6" stop-opacity="0.6" />
                    </linearGradient>
                </defs>
            </svg>
            
            <div class="absolute bottom-4 left-0 w-full flex justify-center space-x-6 text-sm text-slate-400">
                <span class="flex items-center"><span class="w-3 h-3 bg-teal-400 rounded-sm inline-block mr-2 opacity-60"></span> {{ sideA }} w</span>
                <span class="flex items-center"><span class="w-3 h-3 bg-blue-500 rounded-sm inline-block mr-2 opacity-60"></span> {{ sideB }} h</span>
            </div>
        </div>

    </div>
  </div>
</template>

<style scoped>
@keyframes fade-in {
    from { opacity: 0; transform: translateY(10px); }
    to { opacity: 1; transform: translateY(0); }
}
.animate-fade-in {
    animation: fade-in 0.5s ease-out forwards;
}

/* Custom styling for PrimeVue inputs to match our dark theme via deep selectors if needed, 
   though PrimeVue Aura dark theme usually handles this well automatically when configured. */
</style>
