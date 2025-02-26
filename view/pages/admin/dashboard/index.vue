<template>
  <UseHead title="Dashboard - Admin" />

  <div class="h-auto w-full p-5 flex flex-col gap-5">
    <h1 class="text-lg font-semibold text-custom-800 dark:text-white">
      Dashboard
    </h1>

    <div class="font-medium cursor-default dark:text-custom-300">
      Today | {{ today }}
    </div>

    <span
      v-if="
        !user.role || !['client', 'admin', 'superadmin'].includes(user.role)
      "
    >
      <UIcon
        class="animate-spin text-center"
        name="i-heroicons-arrow-path-solid"
      />
      In a moment...
    </span>

    <div
      v-else
      class="grid grid-cols-1 gap-5 w-full h-auto dark:text-custom-400"
    >
      <section class="flex flex-col gap-5">
        <div class="flex items-center w-full gap-5">
          <h1 class="min-w-max">Recently Detected</h1>
          <hr class="border-custom-300 dark:border-custom-500 mx-auto w-full" />
        </div>

        <span v-if="totalActionsDetected <= 0">
          <div class="flex items-center justify-center font-bold">
            0 Detection
          </div>
        </span>
        <span v-else>
          <UCarousel
            ref="carouselRef"
            v-slot="{ item }"
            arrows
            :items="items"
            :ui="{ item: 'basis-full' }"
            :prev-button="{
              color: 'gray',
              icon: 'i-heroicons-arrow-left-20-solid',
            }"
            :next-button="{
              color: 'gray',
              icon: 'i-heroicons-arrow-right-20-solid',
            }"
            class="overflow-hidden w-full h-auto bg-custom-100 dark:bg-custom-900 rounded shadow-md mx-auto relative"
            indicators
          >
            <img :src="item" class="w-[70%] h-auto mx-auto" draggable="false" />
          </UCarousel>
        </span>
      </section>

      <section
        class="grid lg:grid-cols-2 grid-cols-1 gap-5 max-w-full h-auto mt-5"
      >
        <div class="overflow-auto flex flex-col gap-5">
          <div class="flex items-center w-full gap-5">
            <h1 class="min-w-max">Active Accounts</h1>
            <hr
              class="border-custom-300 dark:border-custom-500 mx-auto w-full"
            />
          </div>
          <div
            class="flex flex-col h-full overflow-x-scroll bg-custom-100 dark:bg-custom-900 rounded"
          >
            <div v-if="activeUsers.length > 0" class="flex">
              <NuxtLink
                :to="`/admin/users/${user.username}`"
                v-for="user in activeUsers"
                :key="user.id"
                class="flex flex-col gap-1 items-center cursor-pointer p-5 border border-custom-100 
                hover:border-custom-300 dark:border-custom-900 dark:hover:border-custom-500"
              >
                <div class="relative">
                  <img
                    v-if="user.avatar"
                    draggable="false"
                    class="rounded-full w-14 h-14 border-2 border-custom-400 dark:border-custom-300"
                    :src="user.avatar"
                  />
                  <span
                    v-else
                    class="font-black text-[30px] w-14 h-14 dark:bg-custom-700 bg-custom-400 flex items-center justify-center rounded-full"
                    >{{ user.initial_letter }}</span
                  >
                  <span
                    class="w-5 h-5 flex justify-center items-center dark:bg-green-600 bg-green-700 absolute rounded-full bottom-0 -right-1"
                  >
                    <UIcon
                      class="absolute dark:text-white text-white p-[0.15em]"
                      name="i-lucide-key-round"
                    />
                  </span>
                </div>
                <span class="capitalize text-xs text-center truncate">{{
                  user.name
                }}</span>
                <UKbd
                  class="text-center rounded-full px-2 py-1 bg-custom-700 text-custom-100 dark:text-custom-200 dark:bg-custom-900 dark:border dark:border-custom-500 text-[9px]"
                  :value="user.role"
                />
              </NuxtLink>
            </div>
            <div
              v-else
              class="m-auto cursor-default text-sm text-custom-800 dark:text-custom-500 font-bold"
            >
              Only your account is active
            </div>
          </div>
        </div>
        <div class="overflow-auto flex flex-col gap-5">
          <div class="flex items-center w-full gap-5">
            <h1 class="min-w-max">Inactive Accounts</h1>
            <hr
              class="border-custom-300 dark:border-custom-500 mx-auto w-full"
            />
          </div>
          <div
            class="flex flex-col h-full overflow-x-scroll bg-custom-100 dark:bg-custom-900 rounded"
          >
            <div v-if="inactiveUsers.length > 0" class="flex">
              <NuxtLink
                :to="`/admin/users/${user.username}`"
                v-for="user in inactiveUsers"
                :key="user.id"
                class="flex flex-col gap-1 items-center cursor-pointer p-5 border border-custom-100 
                hover:border-custom-300 dark:border-custom-900 dark:hover:border-custom-500"
              >
                <div class="relative">
                  <img
                    draggable="false"
                    v-if="user.avatar"
                    class="rounded-full w-14 h-14 border-2 border-custom-400 dark:border-custom-300"
                    :src="user.avatar"
                  />
                  <span
                    v-else
                    class="font-black text-[30px] w-14 h-14 dark:bg-custom-700 bg-custom-400 flex items-center justify-center rounded-full"
                    >{{ user.initial_letter }}</span
                  >
                  <span
                    class="w-5 h-5 flex justify-center items-center dark:bg-red-600 bg-red-700 absolute rounded-full bottom-0 -right-1"
                  >
                    <UIcon
                      class="absolute dark:text-white text-white p-[0.15em]"
                      name="i-lucide-lock-keyhole"
                    />
                  </span>
                </div>
                <span class="capitalize text-xs text-center truncate">{{
                  user.name
                }}</span>
                <UKbd
                  class="text-center rounded-full px-2 py-1 bg-custom-700 text-custom-100 dark:text-custom-200 dark:bg-custom-900 dark:border dark:border-custom-500 text-[9px]"
                  :value="user.role"
                />
              </NuxtLink>
            </div>
            <div
              v-else
              class="m-auto cursor-default text-sm text-custom-800 dark:text-custom-500 font-bold"
            >
              No Users Found.
            </div>
          </div>
        </div>
      </section>

      <div class="flex items-center w-full gap-5 mt-5">
        <h1 class="min-w-max">Categories</h1>
        <hr class="border-custom-300 dark:border-custom-500 mx-auto w-full" />
      </div>
      <section
        class="w-full h-auto grid lg:grid-cols-4 sm:grid-cols-2 grid-cols-1 gap-5"
      >
        <NuxtLink
          :to="card.path"
          v-for="card in cards"
          class="p-5 rounded bg-custom-100 dark:bg-custom-900 h-auto flex sm:gap-x-7 gap-x-10 items-center cursor-pointer border dark:border-custom-900 dark:hover:border-custom-500 hover:border-custom-900"
        >
          <div class="rounded-full p-3 bg-custom-600">
            <UIcon class="w-10 h-10 p-1 text-white" :name="card.icon" />
          </div>
          <div
            class="flex flex-col gap-2 justify-start text-custom-600 dark:text-white"
          >
            <h1 class="text-2xl font-bold">{{ card.total }}</h1>
            <p class="text-wrap">{{ card.label }}</p>
          </div>
        </NuxtLink>
      </section>
    </div>
  </div>
</template>

<script setup lang="ts">
import { fetchAvatars, totalItems } from "~/assets/js/avatar";
import {
  fetchUsers,
  fetchActiveUsers,
  fetchInactiveUsers,
} from "~/assets/js/users";
import { detected, fetchAllNotifications } from "~/assets/js/detected";
import { fetchMotions, motions } from "~/assets/js/motions";
import { user } from "~/assets/js/userLogged";

// import { notifications } from "~/assets/js/notifications";
// import { initial } from "~/assets/js/userLogged";

definePageMeta({
  layout: "sidebar",
});

// Fetch the generated user data
const users = ref([]);
const totalUsers = computed(() => users.value.length + 1);
const totalAvatars = totalItems;

const activeUsers = ref([]);
const inactiveUsers = ref([]);

const clients = ref([]);
const admins = ref([]);

const motionsCount = ref([]);
const totalMotions = computed(() => motionsCount.value.length);

const actionsDetected = ref([]);
const totalActionsDetected = computed(() => actionsDetected.value.length);

let activeCount = 0;
let inactiveCount = 0;

const cards = [
  {
    icon: "i-lucide-triangle-alert",
    total: totalActionsDetected,
    label: "Potential Theft",
    path: "/admin/detected"
  },
  {
    icon: "i-lucide-users-round",
    total: totalUsers,
    label: "Total Users",
    path: "/admin/users"
  },
  {
    icon: "i-lucide-user-round-check",
    total: activeCount, // Will be updated after loadUsers runs
    label: "Active Accounts",
    path: { path: '/admin/users?', query: { q: 'active' } }
  },
  {
    icon: "i-lucide-user-round-x",
    total: inactiveCount, // Will be updated after loadUsers runs
    label: "Inactive Accounts",
    path: { path: '/admin/users?', query: { q: 'inactive' } }
  },
  {
    icon: "i-lucide-hand",
    total: totalMotions,
    label: "Motions Detected",
    path: "/admin/motions"
  },
  {
    icon: "i-lucide-smile",
    total: totalAvatars,
    label: "Avatars",
    path: "/admin/avatars"
  },
  {
    icon: "i-lucide-shield-check",
    total: clients,
    label: "Clients",
    path: { path: '/admin/users?', query: { q: 'client' } }
  },
  {
    icon: "i-lucide-circle-user-round",
    total: admins,
    label: "Admins",
    path: { path: '/admin/users?', query: { q: 'admin' } }
  },
];

// fetch screenshots
// Assuming `detected` is correctly imported and contains the necessary data
const items = computed(() => {
  return detected
    .slice(-6)
    .reverse()
    .map((notification) => `/Snapshots/${notification.screenshot}`);
});

const date = new Date();
const today = date.toLocaleString("en-US", { month: "long", day: "numeric" });

const carouselRef = ref();

onMounted(() => {
  fetchAllNotifications();

  fetchAvatars();
  fetchMotions();
  motionsCount.value = motions;
  actionsDetected.value = detected;
  setInterval(() => {
    if (!carouselRef.value) return;

    if (carouselRef.value.page === carouselRef.value.pages) {
      return carouselRef.value.select(0);
    }

    carouselRef.value.next();
  }, 3000);
});

const loadUsers = async () => {
  const fetchedUsers = await fetchUsers();

  let activeCount = 1;
  let inactiveCount = 0;
  let adminCount = 1; // Counter for admins
  let clientCount = 0; // Counter for clients

  // Count active and inactive users, and count admins and clients
  fetchedUsers.forEach((user) => {
    if (user.status === "active") {
      activeCount++;
    } else if (user.status === "inactive") {
      inactiveCount++;
    }

    // Check user role
    if (user.role === "admin" || user.role === "superadmin") {
      adminCount++; // Increment admin counter
    } else if (user.role === "client") {
      clientCount++; // Increment client counter
    }
  });

  // Update the cards array with the new active and inactive counts
  cards.find((card) => card.label === "Active Accounts").total = activeCount;
  cards.find((card) => card.label === "Inactive Accounts").total =
    inactiveCount;

  // Update the clients and admins references
  clients.value = clientCount; // Set the count of clients
  admins.value = adminCount; // Set the count of admins

  console.log(cards);
  return fetchedUsers;
};

onMounted(async () => {
  const fetchedUsers = await loadUsers();
  users.value = fetchedUsers || []; // Ensure users is set to an array

  activeUsers.value = (await fetchActiveUsers()) || [];
  console.log("mao ni listahan sa aktibo", activeUsers);

  inactiveUsers.value = (await fetchInactiveUsers()) || [];
  console.log("listahan sa inactive users", inactiveUsers);
});
</script>
