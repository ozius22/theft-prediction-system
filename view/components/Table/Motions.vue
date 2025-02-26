<template>
  <section class="items-center grid gap-5">
    <div
      class="flex sm:gap-0 gap-5 sm:flex-row flex-col-reverse sm:justify-between justify-center"
    >
      <!-- Search Input -->
      <div class="flex gap-1 justify-start items-end">
        <UInput
          v-model="q"
          name="q"
          placeholder="Search..."
          icon="i-heroicons-magnifying-glass-20-solid"
          autocomplete="off"
          color="gray"
          size="sm"
          :ui="{
            rounded: 'rounded',
            color: {
              gray: { outline: 'dark:bg-custom-100 dark:text-custom-900' },
            },
            icon: { trailing: { pointer: '' } },
          }"
          class="w-full sm:w-auto sm:-mb-0 -mb-5"
        >
          <template #trailing>
            <UButton
              v-show="q !== ''"
              color="gray"
              variant="link"
              icon="i-heroicons-x-mark-20-solid"
              :padded="false"
              @click="q = ''"
              class="hover:text-red-400 dark:hover:text-red-600 text-red-700 dark:text-red-400"
            />
          </template>
        </UInput>
      </div>
      <!-- Pagination -->
      <UPagination
        :model-value="currentPage"
        :page-count="pageCount"
        :total="total"
        :ui="{
          color: 'gray',
          wrapper: 'flex items-center gap-1',
          rounded: '!rounded-full min-w-[30px] justify-center',
          default: {
            activeButton: {
              variant: 'outline',
            },
          },
        }"
        @update:model-value="updatePage"
        class="flex justify-center"
      />
    </div>

    <!-- Motion Table -->
    <UTable
      :columns="tableHeaders"
      :rows="paginatedData"
      sort-asc-icon="i-heroicons-arrow-up"
      sort-desc-icon="i-heroicons-arrow-down"
      class="max-h-[70vh] max-w-full overflow-auto border rounded border-custom-300 dark:border-custom-800"
      :ui="{
        thead:
          'sticky top-0 z-10 dark:bg-custom-700 bg-custom-300 cursor-default',
        tbody: 'bg-custom-100 dark:bg-custom-950',
      }"
    >
      <!-- Custom Template for Row Index -->
      <template #id-data="{ index }">
        <span>{{ (currentPage - 1) * pageCount + index + 1 }}</span>
      </template>

      <template #threshold-data="{ row }">
        <div>
          <span
            v-if="row.threshold <= 74.99"
            class="text-green-500 font-bold"
            >{{ row.threshold }}%</span
          >
          <span v-else class="text-red-500 font-bold"
            >{{ row.threshold }}%</span
          >
      </div>
      </template>

      <!-- Custom Template for Actions -->
      <template #action-data="{ row }">
        <div class="flex justify-start gap-2">
          <UTooltip text="View" :popper="{ arrow: true, placement: 'bottom' }">
            <UIcon
              name="i-lucide-eye"
              class="text-xl hover:opacity-50 text-blue-500"
              @click="viewAction(row)"
            />
          </UTooltip>
          <!-- <UTooltip
            text="Delete"
            :popper="{ arrow: true, placement: 'bottom' }"
          >
            <UIcon
              name="i-lucide-trash-2"
              class="text-xl hover:opacity-50 text-red-500"
              @click="deleteAction(row)"
            />
          </UTooltip> -->
        </div>
      </template>
    </UTable>

    <!-- Pagination -->
    <div class="flex justify-center">
      <span class="text-xs">
        Showing
        <span class="font-medium">{{ startItem }}</span>
        -
        <span class="font-medium">{{ endItem }}</span>
        of
        <span class="font-medium">{{ total }}</span>
        results
      </span>
    </div>
  </section>
</template>

<script setup>
// Imports
import { ModalViewMotions } from "#components";
import { name, playSound } from "~/assets/js/sound";
import { motions, fetchMotions } from "~/assets/js/motions";

// Reactive Data
const currentPage = ref(1);
const pageCount = ref(20);
const q = ref("");

// Table Headers
const tableHeaders = [
  { key: "id", label: "#" },
  { key: "name", label: "Motion" },
  { key: "threshold", label: "Threshold" },
  { key: "action", label: "Action" },
];

// Filtered and Paginated Data
const filteredRows = computed(() => {
  if (!q.value) return motions;
  return motions.filter((motion) =>
    Object.values(motion).some((value) =>
      String(value).toLowerCase().includes(q.value.toLowerCase())
    )
  );
});

// Pagination and Search Logic
const total = computed(() => filteredRows.value.length);
const paginatedData = computed(() => {
  const start = (currentPage.value - 1) * pageCount.value;
  return filteredRows.value.slice(start, start + pageCount.value);
});

// Data and reactive state
const selectedMotion = ref(null); // To store the selected motion

// Open Modal and pass selected motion
const viewAction = (motion) => {
  selectedMotion.value = motion; // Set the selected motion
  const modal = useModal();
  modal.open(ModalViewMotions, { selectedMotion: motion }); // Pass motion as prop
};

// Delete Action
const toast = useToast();

const deleteAction = async (motion) => {
  name.value = "delete_1";

  if (confirm("Delete this motion?")) {
    try {
      const response = await $fetch(
        `http://127.0.0.1:8000/api/motions/${motion.id}`,
        {
          method: "DELETE",
        }
      );

      if (response) {
        selectedMotion.value = await loadMotions();

        playSound();

        toast.add({
          title: "Deleted Successfully!",
          icon: "i-lucide-trash-2",
          timeout: 2000,
          ui: {
            background: "dark:bg-red-700 bg-red-300",
            progress: { background: "dark:bg-white bg-red-700 rounded-full" },
            ring: "ring-1 ring-red-700 dark:ring-custom-900",
            default: { closeButton: { color: "white" } },
            icon: "text-custom-900",
          },
        });

        navigateTo("/admin/motions");
      }
    } catch (error) {
      console.log(error.response);
      console.log("error", error);
      // Assuming you want to set state.errors in case of an error
      state.errors = error.response;
    }
  } else {
    console.log("Cancelled.");
  }
};

// Update Page on Pagination
const updatePage = (page) => (currentPage.value = page);

// Helper Computed Values
const startItem = computed(() => (currentPage.value - 1) * pageCount.value + 1);
const endItem = computed(() =>
  Math.min(currentPage.value * pageCount.value, total.value)
);

// Watch Search Query
watch(q, () => (currentPage.value = 1));

onMounted(async () => {
  const fetchMotions = await loadMotions();
  selectedMotion.value = fetchMotions || "";
  console.log("video path", motions[14].video_path);
});

const loadMotions = async () => {
  await fetchMotions();
  return fetchMotions;
};
</script>
