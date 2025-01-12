<script setup>
import { onMounted, ref } from "vue";
import { Head } from "@inertiajs/vue3";

import AuthenticatedLayout from "@/Layouts/AuthenticatedLayout.vue";

import getGames from "@/services/getGames";
import getWeeklyPlaytimeTotals from "@/services/getWeeklyPlaytimeTotals";
import LineChart from "./Partials/LineChart.vue";

const weeklyTotals = ref({ data: [] });
const xAxisValues = ref([]);
const selectedGame = ref("");
const hasGames = ref(false);
const games = ref([]);

const fetchGames = async () => {
    games.value = await getGames();
    hasGames.value = games.value.length > 0;
};

const getWeeklyTotals = async (gameId) => {
    const { data } = await getWeeklyPlaytimeTotals(gameId);
    const sortedArrayOfWeeklyTotals = Object.entries(data).sort(
        (a, b) => new Date(a[0]) - new Date(b[0])
    );
    const sortedObj = Object.fromEntries(sortedArrayOfWeeklyTotals);
    weeklyTotals.value = {
        name: "Total Playtime (hrs)",
        data: Object.values(sortedObj),
    };
    xAxisValues.value = Object.keys(sortedObj);
};

const onSelectChange = async () => {
    getWeeklyTotals(selectedGame.value.id);
};

onMounted(() => {
    getWeeklyTotals(-1);
    fetchGames();
});
</script>

<template>
    <Head title="Statistics" />

    <AuthenticatedLayout>
        <div class="max-w-4xl mx-auto mt-10">
            <div class="pl-4 pr-4">
                <h1 class="text-2xl font-extrabold text-white mb-5">
                    Game Playtime
                </h1>
                <form class="max-w-sm mt-3">
                    <label
                        for="games"
                        class="block mb-2 text-sm font-medium text-white"
                        >Select a game</label
                    >
                    <select
                        v-model="selectedGame"
                        :disabled="!hasGames"
                        @change="onSelectChange()"
                        id="games"
                        class="disabled:bg-slate-200 border text-sm rounded-lg focus:ring-blue-500 focus:border-blue-500 block w-full p-2.5 bg-gray-700 border-gray-600 placeholder-gray-400 text-white mb-10"
                    >
                        <option selected value="">All games</option>
                        <option v-for="game in games" :value="game">
                            {{ game.title }}
                        </option>
                    </select>
                    <h2 class="text-xl font-extrabold text-white mb-5">
                        {{
                            selectedGame.title
                                ? selectedGame.title
                                : "All Games"
                        }}
                    </h2>
                </form>
            </div>

            <LineChart
                :series="weeklyTotals"
                :xAxis="xAxisValues"
                title="Total Game Playtime (hrs)"
            />
        </div>
    </AuthenticatedLayout>
</template>
