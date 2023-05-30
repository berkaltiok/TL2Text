<template>
  <form class="flex flex-col m-auto max-w-[30rem] w-full bg-white shadow-sm rounded-3xl p-6 gap-6">
    <div class="flex justify-between items-center">
      <div class="text-xl font-bold text-gray-800">Çevirici</div>
      <div class="text-xs font-medium text-gray-500 bg-gray-100 px-2 py-1 rounded-md">Türk Lirası</div>
    </div>
    <div class="flex flex-col gap-2 relative">
      <div
        class="
          absolute
          bg-white
          rounded-full
          border border-gray-200
          m-auto
          w-9
          h-9
          inset-0
          ring-[5px] ring-white
          text-gray-600
          flex
          items-center
          justify-center
        "
      >
        <svg width="20" height="20" viewBox="0 0 40 40" fill="none" xmlns="http://www.w3.org/2000/svg">
          <path
            d="M17 32L17 8L8 17.1035"
            stroke="currentColor"
            stroke-width="2"
            stroke-linecap="round"
            stroke-linejoin="round"
          />
          <path
            d="M23 8L23 32L32 22.8965"
            stroke="currentColor"
            stroke-width="2"
            stroke-linecap="round"
            stroke-linejoin="round"
          />
        </svg>
      </div>
      <label
        class="
          flex flex-col
          justify-center
          border border-gray-200
          rounded-xl
          h-16
          px-3
          hover:border-gray-300
          transition transition-all
        "
      >
        <span class="text-sm font-semibold text-gray-600">Sayı</span>
        <input
          type="text"
          v-model="price"
          class="tracking-tight bg-transparent border-0 h-fit p-0 !outline-none !border-none !ring-0"
          placeholder="0,00"
        />
      </label>
      <label
        class="
          flex flex-col
          justify-center
          border border-gray-200
          rounded-xl
          h-16
          px-3
          hover:border-gray-300
          cursor-alias
          transition transition-all
        "
        @click="toClipboard(result)"
      >
        <span class="text-sm font-semibold text-gray-600">Sonuç</span>
        <input
          disabled
          v-model="result"
          class="tracking-tight bg-transparent border-0 h-fit p-0 !outline-none !border-none !ring-0 h-6 cursor-alias"
          placeholder="SIFIR TÜRK LİRASI"
        />
      </label>
    </div>
  </form>
</template>

<script setup>
  import { ref, computed, watch, watchEffect } from "vue"
  import useClipboard from "vue-clipboard3"

  const { toClipboard } = useClipboard()

  const price = ref("")
  const result = ref("")

  function convertToTurkishCurrencyText(number, space = true) {
    let digits = number
      .toString()
      .replace(/\B(?=(\d{3})+(?!\d))/g, ".")
      .split(",")
    let integerPart = digits[0]
    let decimalPart = digits[1] || ""

    const ones = ["", "Bir", "İki", "Üç", "Dört", "Beş", "Altı", "Yedi", "Sekiz", "Dokuz"]
    const tens = ["", "On", "Yirmi", "Otuz", "Kırk", "Elli", "Altmış", "Yetmiş", "Seksen", "Doksan"]
    const thousands = ["", "Bin", "Milyon", "Milyar", "Trilyon", "Katrilyon", "Kentrilyon"]

    let result = []
    let step = 0

    for (let i = integerPart.split(".").length; i > 0; i--) {
      let part = integerPart.split(".")[i - 1]

      if (part.length == 1) part = "00" + part
      if (part.length == 2) part = "0" + part

      let text = ""

      for (let j = 1; j < part.length + 1; j++) {
        if (j == 1 && part[j - 1] == 1) text += " Yüz "
        else if (j == 1 && ones[part[j - 1]] != "") text += ones[part[j - 1]] + " Yüz "
        else if (j == 2) text += tens[part[j - 1]] + " "
        else if (j == 3 && integerPart.length == 5 && part[j - 1] == 1 && step == 1) text += " "
        else if (j == 3) text += ones[part[j - 1]] + " "
      }

      if (text != "") result.push(text + thousands[step])

      step++
    }

    if (result.length != 0) result = result.reverse().join(" ") + "Türk Lirası"
    else result = ""

    if (decimalPart.length == 1) decimalPart = decimalPart + "0"
    if (decimalPart != "") result += " " + tens[decimalPart[0]] + " " + (ones[decimalPart[1]] !== "" ? ones[decimalPart[1]] + " Kuruş" : "")

    result = result.replace(/ /g, space ? " " : "").trim()

    return `#Yalnız ${result}#`
  }

  watch(
    () => price.value,
    () => {
      result.value = convertToTurkishCurrencyText(price.value, false).toLocaleUpperCase("tr-TR")
    }
  )
</script>
