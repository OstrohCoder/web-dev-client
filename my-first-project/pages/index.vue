<template>
  <div>
    <div class="flex flex-col items-center">
      <h1 class="text-center text-3xl mt-10 mb-5">Список продуктів</h1>
      <UInput v-model="globalFilter" class="max-w-sm mb-4" placeholder="Шукати..." />
    </div>
    <div>
      <p v-if="status === 'pending'" class="mb-4 text-gray-500">Завантаження...</p>
      <p v-else-if="status === 'error'" class="mb-4 text-red-500">Сталася помилка під час завантаження</p>
      <UTable
          v-else
          ref="table"
          v-model:sorting="sorting"
          v-model:global-filter="globalFilter"
          v-model:pagination="pagination"
          :data="data || []"
          :columns="columns"
          :pagination-options="{ getPaginationRowModel: getPaginationRowModel() }"
          class="flex-1"
      />
    </div>
    <div class="flex justify-center border-t border-(--ui-border) pt-4">
      <UPagination
          :default-page="(table?.tableApi?.getState().pagination.pageIndex || 0) + 1"
          :items-per-page="table?.tableApi?.getState().pagination.pageSize"
          :total="table?.tableApi?.getFilteredRowModel().rows.length"
          @update:page="(p) => table?.tableApi?.setPageIndex(p - 1)"
      />
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, h, watch } from 'vue'
import { useHead } from '#imports'
import type { TableColumn } from '@nuxt/ui'
import { getPaginationRowModel } from '@tanstack/vue-table'

useHead({
  title: 'Список продуктів',
})

const table = ref()
const globalFilter = ref('')
const pagination = ref({
  pageIndex: 0,
  pageSize: 5
})

watch(globalFilter, () => {
  pagination.value.pageIndex = 0
})

type Product = {
  title: string
  description: string
  price: number
  rating: number
  brand: string
  category: string
  thumbnail: string
}

const { data, status } = await useFetch<Product[]>('https://dummyjson.com/products', {
  key: 'table-products',
  transform: (data: any) => data?.products || [],
  lazy: true
})

const columns: TableColumn<Product>[] = [
  {
    accessorKey: 'title',
    header: 'Назва'
  },
  {
    accessorKey: 'description',
    header: 'Опис',
    cell: ({ row }) =>
        h('div', { class: 'text-sm max-w-[100%] whitespace-normal break-words overflow-hidden' }, row.getValue('description'))
  },
  {
    accessorKey: 'price',
    header: () => h('div', { class: 'text-right' }, 'Ціна'),
    cell: ({ row }) => {
      const price = Number.parseFloat(row.getValue('price'))

      const formatted = new Intl.NumberFormat('uk-UA', {
        style: 'currency',
        currency: 'UAH'
      }).format(price)

      return h('div', { class: 'text-right font-medium' }, formatted)
    }
  },
  {
    accessorKey: 'rating',
    header: 'Оцінка',
    cell: ({ row }) => {
      const rating: number = row.getValue('rating')
      const colorClass = rating >= 4.5 ? 'text-green-500' : 'text-red-500'

      return h('span', { class: colorClass }, `${rating} ★`)
    }
  },
  {
    accessorKey: 'brand',
    header: 'Бренд',
    cell: ({ row }) => h('span', { class: 'capitalize' }, row.getValue('brand'))
  },
  {
    accessorKey: 'category',
    header: 'Категорія',
    cell: ({ row }) => h('span', { class: 'capitalize' }, row.getValue('category'))
  },
  {
    accessorKey: 'thumbnail',
    header: 'Фото',
    cell: ({ row }) =>
        h('div', {
          class: 'w-[100px] h-[100px] overflow-hidden'
        }, [
          h('img', {
            src: row.getValue('thumbnail'),
            alt: 'Product Image',
            class: 'w-full h-full object-cover'
          })
        ])
  },
]

const sorting = ref([
  {
    id: 'title',
    desc: false
  }
])
</script>
