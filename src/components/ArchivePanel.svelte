<script lang="ts">
import { onMount } from "svelte";

import { getPostUrlBySlug } from "../utils/url-utils";

export let sortedPosts: Post[] = [];

interface Post {
	slug: string;
	data: {
		title: string;
		startDate: Date;
		endDate: Date;
	};
}

interface Group {
	year: number;
	posts: Post[];
}

let groups: Group[] = [];

function formatDate(date: Date, includeYear = false) {
	const month = (date.getMonth() + 1).toString().padStart(2, "0");
	const day = date.getDate().toString().padStart(2, "0");
	return includeYear ? `${month}-${day}-${date.getFullYear()}` : `${month}-${day}`;
}

function formatDateRange(startDate: Date, endDate: Date) {
	const endIncludesYear = startDate.getFullYear() !== endDate.getFullYear();
	return `${formatDate(startDate)} - ${formatDate(endDate, endIncludesYear)}`;
}

onMount(async () => {
	const grouped = sortedPosts.reduce(
		(acc, post) => {
			const year = post.data.startDate.getFullYear();
			if (!acc[year]) {
				acc[year] = [];
			}
			acc[year].push(post);
			return acc;
		},
		{} as Record<number, Post[]>,
	);

	const groupedPostsArray = Object.keys(grouped).map((yearStr) => ({
		year: Number.parseInt(yearStr, 10),
		posts: grouped[Number.parseInt(yearStr, 10)],
	}));

	groupedPostsArray.sort((a, b) => b.year - a.year);

	groups = groupedPostsArray;
});
</script>

<div class="card-base px-8 py-6">
    {#each groups as group}
        <div>
            <div class="flex flex-row w-full items-center h-[3.75rem]">
                <div class="w-[35%] md:w-[25%] transition text-2xl font-bold text-right text-75">
                    {group.year}
                </div>
                <div class="w-[10%] md:w-[8%]">
                    <div
                            class="h-3 w-3 bg-none rounded-full outline outline-[var(--primary)] mx-auto
                  -outline-offset-[2px] z-50 outline-3"
                    ></div>
                </div>
                <div class="w-[55%] md:w-[67%] transition text-left text-50">
                    {group.posts.length} {group.posts.length === 1 ? "post" : "posts"}
                </div>
            </div>

            {#each group.posts as post}
                <a
                        href={getPostUrlBySlug(post.slug)}
                        aria-label={post.data.title}
                        class="group btn-plain !block h-10 w-full rounded-lg hover:text-[initial]"
                >
                    <div class="flex flex-row justify-start items-center h-full">
                        <!-- date -->
                        <div class="w-[35%] md:w-[25%] transition text-xs text-right text-50 whitespace-nowrap">
                            {formatDateRange(post.data.startDate, post.data.endDate)}
                        </div>

                        <!-- dot and line -->
                        <div class="w-[10%] md:w-[8%] relative dash-line h-full flex items-center">
                            <div
                                    class="transition-all mx-auto w-1 h-1 rounded group-hover:h-5
                       bg-[oklch(0.5_0.05_var(--hue))] group-hover:bg-[var(--primary)]
                       outline outline-4 z-50
                       outline-[var(--card-bg)]
                       group-hover:outline-[var(--btn-plain-bg-hover)]
                       group-active:outline-[var(--btn-plain-bg-active)]"
                            ></div>
                        </div>

                        <!-- post title -->
                        <div
                                class="w-[55%] md:w-[67%] text-left font-bold
                     group-hover:translate-x-1 transition-all group-hover:text-[var(--primary)]
                     text-75 pr-8 whitespace-nowrap overflow-ellipsis overflow-hidden"
                        >
                            {post.data.title}
                        </div>

                    </div>
                </a>
            {/each}
        </div>
    {/each}
</div>
