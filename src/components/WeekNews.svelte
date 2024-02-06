<script>
	import { onMount } from 'svelte';
    import yaml from 'js-yaml';
    import { marked } from 'marked';
    import { setWeek, nextMonday, format, addDays } from 'date-fns';
    import * as config from '../config.yaml';

    let data = {};
    let cats = [];

    export let year = "2024";
    export let week = "05";
    export let current = false;

    onMount(async () => {
		const res = await fetch(`https://raw.githubusercontent.com/web3privacy/news/main/src/${year}/week${week}.yaml`);
		const yamlData = await res.text();
        data = yaml.load(yamlData);
        if (!data.news) {
            return null
        }
        let uncat = false
        for (const item of data.news) {
            if (item.cat && !cats.includes(item.cat)) {
                cats = [...cats, item.cat]
            }
            if (!item.cat) {
                uncat = true
            }
        }
        if (uncat) {
            cats = [...cats, "uncategorized"]
        }
	});

    const weekStart = setWeek(nextMonday(new Date(Number(year), 0, 4)), Number(week), {
        weekStartsOn: 1,
        firstWeekContainsDate: 4,
    });
    const weekEnd = addDays(weekStart, 6);
    //const weekStartDa

    function capitalizeFirstLetter(string) {
        return string.charAt(0).toUpperCase() + string.slice(1);
    }

</script>

<div class="mb-8 border {current ? "border-dashed border-blue-950" : "border-white/20"}">
    
    <div class="flex w-full p-4 sm:p-6 {current ? "bg-blue-950" : "bg-white/10"}">
        <div class="grid gap-2 sm:gap-6 sm:flex items-center">
            <h1 class="text-2xl">Week {week}/{year}</h1>
            <div class="text-xl {current ? "text-white/50" : ""}">{format(weekStart, "MMM d")} - {format(weekEnd, "MMM d, yyyy")}</div>
            {#if current}
                <div class="text-white">🚧 Current week</div>
            {/if}
        </div>
        <div class="grow"></div>
        <div class="">
            <a href="https://github.com/web3privacy/news/edit/main/src/{year}/week{week}.yaml">Edit</a>
        </div>
    </div>

    {#if data?.news}
        <div class="p-4 sm:p-6">
            {#each cats as cat}
                <div class="mb-6">
                    <h2 class="mb-4 text-xl">{config.cats[cat] ? config.cats[cat].title : capitalizeFirstLetter(cat)}</h2>
                    <ul class="list-disc">
                        {#each data.news.filter(n => (cat === "uncategorized" ? !n.cat : n.cat === cat)) as item}
                            <li class="ml-6 news-item mb-2">
                                {@html marked.parseInline(item.text)}
                                {#if item.src}
                                    <span class="news-item-source text-white/20">(<a href={item.src} class="text-white/20">src</a>)</span>
                                {/if}
                            </li>
                        {/each}
                    </ul>
                </div>
            {/each}
        </div>

        <!--div class="mt-6 text-sm p-4 bg-white/10">
            <div>News count: {data.news?.length}</div>
            <div>Categories: {cats.join(', ')}</div>
        </div-->
    {:else}
        <div class="mt-4 p-6">Loading week ..</div>
    {/if}
</div>