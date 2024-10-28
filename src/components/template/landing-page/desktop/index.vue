<template>
  <div>
    <Breadcrumb :list="['Home', landingPage?.header_text ?? '']" />

    <div class="mt-5">
      <ProductTitle 
        :image="gameInfo?.game.og_image_url ?? ''" 
        :title="landingPage?.header_text ?? ''" 
        :description="landingPage?.meta_description ?? ''"
      />
    </div>

    <div class="grid grid-cols-4 gap-4 mt-5">
      <CardProduct
        v-for="product in products"
        :key="product.id"
        :productImage="forceHttp(product.image_url ?? '', BLUR)"
        :productName="product.name"
        :productOriginalPrice="currencyHelper(
          exchangeRate, 
          regionData,
          product.competitor_price && product.competitor_price > product.price ? product.competitor_price : product.price,
          currencySymbol ? currencySymbol[regionData?.country_code ?? 'ID'] : ''
        )"
        :productDiscount="product.discount ? `${Math.round(product.discount)}%` : undefined"
        :productAdjustmentPrice="currencyHelper(
          exchangeRate,
          regionData,
          product.price,
          currencySymbol ? currencySymbol[regionData?.country_code ?? 'ID'] : ''
        )"
        :averageDeliveryTime="product.average_delivery_time ? formatDeliveryTime(product.average_delivery_time) : ''"
        :soldCount="product.order_count"
        :deliveryType="product.use_instant_delivery ? DeliveryType.INSTANT : (product.use_fast_delivery ? DeliveryType.FAST : DeliveryType.ALL)"
        :isAds="product.use_ads"
      />
    </div>

    <div v-if="isFetchProductLoading" class="grid grid-cols-4 gap-4 mt-5">
      <CardProductLoading v-for="i in 4" :key="i" />
    </div>

    <!-- <div class="my-5">Count {{ count }}</div> -->

    <div class="w-full text-center my-10 relative p-1">
      <div class="absolute inset-x-0 top-1/2 transform -translate-y-1/2 flex items-center">
        <div class="flex-grow border-t border-gray-300"></div>
        <span class="mx-4 bg-white px-4 text-blue-500 flex items-center cursor-pointer" @click="onClickLoadMore">
          Load More
          <!-- <svg class="w-4 h-4 text-blue-500 ml-2 transition-transform duration-300 transform" :class="{ 'rotate-180': isOpen }" fill="currentColor" viewBox="0 0 20 20">
            <path fill-rule="evenodd" d="M5.293 7.293a1 1 0 011.414 0L10 10.586l3.293-3.293a1 1 0 111.414 1.414l-4 4a1 1 0 01-1.414 0l-4-4a1 1 0 010-1.414z" clip-rule="evenodd" />
          </svg> -->
        </span>
        <div class="flex-grow border-t border-gray-300"></div>
      </div>
    </div>
  </div>
</template>
<script setup lang="ts">
import { toRefs, ref } from 'vue';
import type { ILandingPageProps } from '@/types/landing-page';
import Breadcrumb from "@/components/common/Breadcrumb.vue";
import ProductTitle from "@/components/landing-page/ProductTitle.vue";
import CardProduct from "@/components/landing-page/CardProduct.vue";
import CardProductLoading from "@/components/landing-page/CardPorudctLoading.vue";
import { currencyHelper, forceHttp } from "@/utils/common-helper";
import { DeliveryType } from "@/types/delivery";
import ApiHelper from '@/utils/api';
import { REGION_CODE } from '@/types/region';
import { productListHelper } from '@/utils/product-helper';

const BLUR = 'data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVR42mOUlZKqBwABfwDSiUgXoQAAAABJRU5ErkJggg==';

const props = defineProps<{
  landingPageProps: ILandingPageProps
}>()

const {
  landingPage,
  gameInfo,
  products,
  exchangeRate,
  regionData,
  currencySymbol,
  configReducersReplica,
} = toRefs(props.landingPageProps)

const isFetchProductLoading = ref(false)

const formatDeliveryTime = (deliveryTime: number) => {
  const hours = Math.floor(deliveryTime / 60);
  const minutes = Math.floor(deliveryTime % 60);
  return `${hours > 0 ? `${hours} jam ` : ''}${minutes} menit`;
}

const onClickLoadMore = () => {
  fetchProduct()
}

const fetchProduct = async () => {
  isFetchProductLoading.value = true

  const params = {
    is_include_game: 1,
    is_include_item_type: 1,
    is_include_item_info_group: 0,
    is_include_order_record: 1,
    is_from_web: 1,
    exclude_sharing_account_eligible: 1,
    is_include_upselling_product: 1,
    use_simple_pagination: 1,
    is_auto_delivery_first: 1,
    is_with_promotion: 1,
    is_include_instant_delivery: 1,
    is_default_product_list: 1,
    is_enough_stock: 1,
    page: 1,
    per_page: 48,
    sort: 'popular',
    ...(landingPage?.value?.game_id !== undefined && { game_id: landingPage.value.game_id }),
    ...(landingPage?.value?.item_type_id !== undefined && { item_type_id: landingPage.value.item_type_id }),
    ...(landingPage?.value?.item_info_id !== undefined && { item_info_id: landingPage.value.item_info_id }),
    is_exclusive: false,
    country_codes: ['ID'],
  };

  try {
    const response = await ApiHelper.get('/v1/product', params);
    if (response.success) {
      const formattedProducts = productListHelper(response.data.data, configReducersReplica?.value, REGION_CODE.INDONESIA);
      products.value = [
        ...products.value, 
        ...formattedProducts,
      ]
    } else {
      alert(`Error fetching products, ${response.message}`);
    }
  } catch (error) {
    console.error('Error fetching products:', error);
  } finally {
    isFetchProductLoading.value = false;
  }
}

</script>
