<template>
    <div class="page-layout">
        <AppSidebar
            :class="['page-layout__sidebar', menuOpened ? 'page-layout__sidebar--opened' : '']"
            @closeMenu="burgerToggleHandler"
        />
        <main class="page-layout__main-page">
            <div class="page-layout__container">
                <h2 class="page-layout__heading">
                    Главная
                </h2>
                <TheStoriesLayout class="page-layout__stories-layout"/>
                <TheNavigationLayout class="page-layout__navigation-layout"/>
                <TheModulesLayout class="page-layout__modules-layout"/>
                <ThePersonalLayout class="page-layout__personal-layout"/>
                <TheRecommendedMediaLayout class="page-layout__media-layout"/>
            </div>
            <aside class="page-layout__aside">
                <AppUserNavigation class="page-layout__user-navigation"/>
                <TheCalendar
                    :event-dates="eventDatesList"
                    class="mt-8"
                    @on-date-clicked="getDate($event)"
                />
                <TheUpcomingEvents class="page-layout__upcoming-events"/>
            </aside>
        </main>
        <AppMobileMenu
            class="page-layout__mobile-menu"
            @burger-toggle="burgerToggleHandler"
        />
    </div>
</template>

<script setup>
import { ref } from 'vue';

import AppSidebar from '@/components/AppSidebar.vue';
import TheStoriesLayout from '@/components/TheStoriesLayout.vue';
import TheNavigationLayout from '@/components/TheNavigationLayout.vue';
import TheModulesLayout from '@/components/TheModulesLayout.vue';
import ThePersonalLayout from '@/components/ThePersonalLayout.vue';
import TheRecommendedMediaLayout from '@/components/TheRecommendedMediaLayout.vue';
import AppUserNavigation from '@/components/AppUserNavigation.vue';
import TheUpcomingEvents from '@/components/TheUpcomingEvents.vue';
import AppMobileMenu from '@/components/AppMobileMenu.vue';
import TheCalendar from '@/components/TheCalendar.vue';

const menuOpened = ref(false);
const burgerToggleHandler = () => menuOpened.value = !menuOpened.value; // maybe will be more logic

const getDate = (date) => console.log(date); // waiting backend
const eventDatesList = ['2023-07-30', '2023-07-31', '2023-08-03', '2023-08-10', '2023-08-11', '2023-08-18', '2023-08-21', '2023-08-29', '2023-09-01', '2023-09-03', '2023-09-12', '2023-09-14'];
</script>

<style>
.page-layout {
    position: relative;
    margin: 0 auto;
    padding: 0;
    flex-grow: 1;
    display: grid;
    grid-template-columns: 280px auto;
}

.page-layout__sidebar {
}

.page-layout__main-page {
    box-sizing: border-box;
    padding: 32px 60px 60px;
    grid-column: 2 / 3;
    display: grid;
    grid-template-columns: auto 470px;
    grid-gap: 30px;
}

.page-layout__container {
    margin: 0;
    min-width: 0;
}

.page-layout__heading {
    margin: 0;
    font-family: var(--headings-font);
    font-weight: 700;
    font-style: normal;
    font-size: 32px;
    line-height: 40px;
    color: var(--headings-color);
    text-transform: uppercase;
}

.page-layout__stories-layout,
.page-layout__navigation-layout,
.page-layout__modules-layout,
.page-layout__personal-layout,
.page-layout__media-layout,
.page-layout__upcoming-events {
    margin: 32px 0 0;
}

.page-layout__aside {
    margin: 0;
    display: flex;
    flex-direction: column;
    justify-content: flex-start;
    align-items: flex-start;
}

.page-layout__user-navigation {
    margin: 0 0 0 auto;
}

.page-layout__mobile-menu {
    position: fixed;
    left: 0;
    bottom: 0;
    right: 0;

    z-index: 2;
}

@media screen and (max-width: 1920px) {
    .page-layout__main-page {
        grid-template-columns: 66fr 32fr;
    }
}

@media screen and (max-width: 1600px) {
    .page-layout__main-page {
        padding: 32px 30px 60px;
        grid-template-columns: 1fr 300px;
    }
}

@media screen and (max-width: 1440px) {
    .page-layout__main-page {
        padding: 32px 30px 60px;
        grid-template-columns: 1fr 300px;
    }
}

@media screen and (max-width: 1366px) {
    .page-layout__main-page {
        padding: 32px 30px 60px;
        grid-template-columns: 1fr;
    }

    .page-layout__user-navigation {
        display: none;
    }
}

@media screen and (max-width: 1024px) {
    .page-layout {
        grid-template-columns: 1fr;
    }

    .page-layout__sidebar {
        position: fixed;
        z-index: 3;

        display: none;
    }

    .page-layout__sidebar--opened {
        display: flex;
    }
}
</style>
