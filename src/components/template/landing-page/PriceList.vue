<template>
  <div class='flex flex-col space-y-4 max-w-[640px]' :class="classStyle">
    <PriceList
      v-if="products && products.length > 0"
      :title="`Daftar Harga ${itemType?.name} Terbaru ${MONTH_NAME[new Date().getMonth()]} ${new Date().getFullYear()}`"
      :list="products.slice(0, MAX_SEO_PRICE_LIST_PRODUCT).map((product: IProduct) => {
        let priceText = currencyHelper(
          exchangeRate,
          regionData,
          product.price || 0,
          mapCurrencySymbol(currencySymbol, regionData?.country_currency),
        );
        return {
          name: product.name,
          price: priceText,
        }
      })"
    />
    <PriceList
      v-if="priceInfos && priceInfos.length > 0"
      :title="`Daftar Harga ${itemType?.name} Terbaru ${new Date().getDate()}/${new Date().getMonth()}/${new Date().getFullYear()}`"
      :list="priceInfos.map((product: IPriceList) => {
        let priceText = currencyHelper(
          exchangeRate,
          regionData,
          product.price || 0,
          mapCurrencySymbol(currencySymbol, regionData?.country_currency),
        );
        return {
          name: product.name,
          price: priceText,
        }
      })"
    />
  </div>
</template>

<script setup lang="ts">
import type { IProduct } from '@/types/product';
import { toRefs } from 'vue';
import { currencyHelper, mapCurrencySymbol } from "@/utils/common-helper";
import PriceList from "@/components/landing-page/PriceList.vue";
import { MONTH_NAME } from "@/constants/date";
import type { IRegionData } from '@/types/region';
import type { IApiForeignExchangeRate } from '@/types/foreign-exchange';
import type { IGameInfoItemType } from '@/types/game';
import type { ICurrencySymbol } from '@/types/currency';
import type { IPriceList } from '@/types/landing-page';

const MAX_SEO_PRICE_LIST_PRODUCT = 10;

const props = defineProps<{
  classStyle?: string
  products: IProduct[],
  exchangeRate: IApiForeignExchangeRate | undefined,
  regionData: IRegionData | undefined,
  currencySymbol: ICurrencySymbol | undefined,
  itemType: IGameInfoItemType | undefined,
  priceInfos: IPriceList[],
}>()

const { classStyle, products, exchangeRate, regionData, currencySymbol, itemType } = toRefs(props)

</script>
