<template>
    <table class="calendar-month__table" role="application" @focusin="onFocusin" @focusout="onFocusout">
        <colgroup>
            <col v-if="!internalHideWeekNumbers" class="calendar-month__col--week" />
            <col class="calendar-month__col--day" />
            <col class="calendar-month__col--day" />
            <col class="calendar-month__col--day" />
            <col class="calendar-month__col--day" />
            <col class="calendar-month__col--day" />
            <col class="calendar-month__col--day" />
            <col class="calendar-month__col--day" />
        </colgroup>

        <thead role="presentation">
            <tr role="presentation">
                <th
                    v-if="!internalHideWeekNumbers"
                    scope="col"
                    aria-hidden="true"
                    class="calendar-month__header-cell"
                ></th>

                <th
                    v-for="weekday in weekdays"
                    :key="weekday.name"
                    scope="col"
                    role="presentation"
                    class="calendar-month__header-cell"
                >
                    <abbr aria-hidden="true" :title="weekday.name">{{
                        showShortWeekdays ? weekday.shortName : weekday.name
                    }}</abbr>
                </th>
            </tr>
        </thead>

        <tbody role="presentation">
            <tr v-for="week in weeks" :key="week.week" role="presentation">
                <td
                    v-if="!internalHideWeekNumbers"
                    class="calendar-month__cell calendar-month__cell--week-number"
                    aria-hidden="true"
                >
                    {{ week.week }}
                </td>

                <td
                    v-if="getDayStartOffset(week.days)"
                    class="calendar-month__cell"
                    :colspan="getDayStartOffset(week.days)"
                    aria-hidden="true"
                ></td>

                <td v-for="day in week.days" :key="day.toString()" class="calendar-month__cell" role="presentation">
                    <!-- @slot Slot for rendering of day content.

Slot scope is available through `v-slot="{ <propertyName> }"`, e.g.
`v-slot="{ date, focused }"`.

The following properties are available:

* `date: FDate;` Date of day.
* `focused: bool;` If month component is focused or not.
        -->
                    <slot :date="day" :focused="focused"></slot>
                </td>

                <td
                    v-if="getDayEndOffset(week.days)"
                    class="calendar-month__cell"
                    :colspan="getDayEndOffset(week.days)"
                    aria-hidden="true"
                ></td>
            </tr>

            <tr v-if="weeks.length < 5" aria-hidden="true">
                <td class="calendar-month__cell" :colspan="totalCols" aria-hidden="true"></td>
            </tr>

            <tr v-if="weeks.length < 6" aria-hidden="true">
                <td class="calendar-month__cell" :colspan="totalCols" aria-hidden="true"></td>
            </tr>
        </tbody>
    </table>
</template>

<script lang="ts">
import { defineComponent, type PropType } from "vue";
import { FDate, groupByWeek, getWeekdayNamings } from "@fkui/date";
import { debounce } from "@fkui/logic";
import { getDayStartOffset, getDayEndOffset } from "./get-day-offset";

export default defineComponent({
    name: "ICalendarMonthGrid",
    props: {
        /**
         * Focused month.
         * @model
         */
        value: {
            type: Object as PropType<FDate>,
            required: true,
        },
        /**
         * Hide week numbers.
         */
        hideWeekNumbers: {
            type: Boolean,
            required: false,
            default: false,
        },
    },
    data() {
        return {
            weekdays: getWeekdayNamings(),
            focused: false,
            resizeObserver: undefined as ResizeObserver | undefined,
            internalHideWeekNumbers: false,
            showShortWeekdays: false,
        };
    },
    computed: {
        totalCols(): number {
            return this.hideWeekNumbers ? 7 : 8;
        },
        weeks(): Array<{ week: number; days: FDate[] }> {
            return groupByWeek(this.value.startOfMonth(), this.value.endOfMonth());
        },
    },
    mounted() {
        this.resizeObserver = new ResizeObserver(debounce(this.onResize, 100));
        this.resizeObserver.observe(this.$el);
        this.onResize();
    },
    unmounted() {
        if (this.resizeObserver) {
            this.resizeObserver.disconnect();
        }
    },
    methods: {
        onResize(): void {
            const component = this.$el as HTMLElement;
            this.internalHideWeekNumbers = this.hideWeekNumbers || component.offsetWidth < 320;
            this.showShortWeekdays = component.offsetWidth < 640;
        },
        onFocusin() {
            this.focused = true;
        },
        onFocusout(e: FocusEvent) {
            const component = this.$el as HTMLElement;
            const relatedTarget = e.relatedTarget as Node;

            if (!component.contains(relatedTarget)) {
                this.focused = false;
            }
        },
        getDayStartOffset(days: FDate[]): number {
            return getDayStartOffset(days);
        },
        getDayEndOffset(days: FDate[]): number {
            return getDayEndOffset(days);
        },
    },
});
</script>
