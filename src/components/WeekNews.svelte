<script>
	import { onMount } from 'svelte';
    import yaml from 'js-yaml';
    import { marked } from 'marked';

    let data = {};
    let cats = [];

    export let year = "2024";
    export let week = "05";

    onMount(async () => {
		const res = await fetch(`https://raw.githubusercontent.com/web3privacy/news/main/src/${year}/week${week}.yaml`);
		const yamlData = await res.text();
        data = yaml.load(yamlData);
        if (!data.news) {
            return null
        }
        for (const item of data.news) {
            if (item.cat && !cats.includes(item.cat)) {
                cats = [...cats, item.cat]
            }
        }
	});

</script>

<div class="mb-14 border border-white/20 p-6">
    
    <div class="flex w-full">
        <h1 class="text-3xl">Week {week}/{year}</h1>
        <div class="grow"></div>
        <div>
            <a href="https://github.com/web3privacy/news/edit/main/src/{year}/week{week}.yaml">Edit</a>
        </div>
    </div>

    {#if data?.news}

        <div>
            {#each cats as cat}
                <div>
                    <h2 class="mt-6 mb-4 text-xl">{cat}</h2>
                    <ul class="list-disc">
                        {#each data.news.filter(n => n.cat === cat) as item}
                            <li class="ml-6 news-item">
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

        <div class="mt-6 text-sm">
            <div>News count: {data.news?.length}</div>
            <div>Categories: {cats.join(', ')}</div>
        </div>
    {:else}
        <div class="mt-4">Loading week ..</div>
    {/if}
</div>