---
title: 如何为你的fuwari博客添加置顶项
published: 2026-01-12
description: ''
image: ''
tags: [教程]
category: ''
draft: false 
lang: ''
---

## 第一步：向 'src/content/config.ts' 中添加一个 pinned 字段

~~~typescript
// src/content/config.ts
import { defineCollection, z } from 'astro:content';

const posts = defineCollection({
  schema: z.object({
    title: z.string(),
    published: z.date(),
    // ... 其他原有的字段 ...
    
    //  添加下面这行：
    pinned: z.boolean().optional().default(false), 
  }),
});

export const collections = { posts };
~~~
## 第二步：修改排序逻辑 (content-utils.ts)

我们将排序逻辑改为：先比较置顶状态，再比较发布时间。

~~~typescript
// src/utils/content-utils.ts

// ... 前面的 imports 保持不变

// Retrieve posts and sort them by pinned status then by publication date
async function getRawSortedPosts() {
	const allBlogPosts = await getCollection("posts", ({ data }) => {
		return import.meta.env.PROD ? data.draft !== true : true;
	});

	const sorted = allBlogPosts.sort((a, b) => {
		// 1. 获取置顶状态 (如果没有设置，默认为 false)
		const pinA = a.data.pinned ?? false;
		const pinB = b.data.pinned ?? false;

		// 2. 如果置顶状态不同，置顶的排在前面
		if (pinA !== pinB) {
			return pinA ? -1 : 1;
		}

		// 3. 如果置顶状态相同（都置顶或都不置顶），则按时间倒序
		const dateA = new Date(a.data.published);
		const dateB = new Date(b.data.published);
		return dateA > dateB ? -1 : 1;
	});

	return sorted;
}

// ... 后面的 export 函数保持不变
~~~

## 第三步：修改 UI 显示 (PostCard.astro)

为了让置顶文章一目了然，我们要在标题旁边加一个“图钉”图标。  
修改 PostCard.astro，在显示 {title} 的位置前面添加图标判断逻辑。  
找到代码中 <a href={url} ...> 包含 {title} 的部分，修改如下：
~~~astro
<!-- src/components/PostCard.astro -->

<!-- ... 前面的代码保持不变 ... -->

<a href={url}
   class="transition group w-full block font-bold mb-3 text-3xl text-90
hover:text-[var(--primary)] dark:hover:text-[var(--primary)]
active:text-[var(--title-active)] dark:active:text-[var(--title-active)]
before:w-1 before:h-5 before:rounded-md before:bg-[var(--primary)]
before:absolute before:top-[35px] before:left-[18px] before:hidden md:before:block
">
    <!--  新增代码开始：如果置顶，显示图标 -->
    {entry.data.pinned && (
        <Icon 
            name="material-symbols:push-pin" 
            class="inline-block mr-2 text-[var(--primary)] align-middle text-2xl mb-1 rotate-45" 
            aria-label="Pinned Post"
        />
    )}
    <!--  新增代码结束 -->

    {title}
    
    <Icon class="inline text-[2rem] text-[var(--primary)] md:hidden translate-y-0.5 absolute" name="material-symbols:chevron-right-rounded" ></Icon>
    <Icon class="text-[var(--primary)] text-[2rem] transition hidden md:inline absolute translate-y-0.5 opacity-0 group-hover:opacity-100 -translate-x-1 group-hover:translate-x-0" name="material-symbols:chevron-right-rounded"></Icon>
</a>

<!-- ... 后面的代码保持不变 ... -->
~~~
## 第四步：如何使用

现在功能已经做好了，只要在你想要置顶的文章的 Markdown 头部（Frontmatter）添加 pinned: true 即可：
~~~markdown
---
title: 我的置顶文章
published: 2024-01-01
description: 这是最重要的文章
tags: [重要]
pinned: true
---
~~~
## 注意事项

如果 'pnpm build' 报错可以尝试执行
~~~bash
pnpm add -D @iconify-json/material-symbols
~~~

## 完整文件

**1. src/content/config.ts**
~~~typescript
import { defineCollection, z } from "astro:content";

const postsCollection = defineCollection({
	schema: z.object({
		title: z.string(),
		published: z.date(),
		updated: z.date().optional(),
		draft: z.boolean().optional().default(false),
		description: z.string().optional().default(""),
		image: z.string().optional().default(""),
		tags: z.array(z.string()).optional().default([]),
		category: z.string().optional().nullable().default(""),
		lang: z.string().optional().default(""),

		/* For internal use */
		prevTitle: z.string().default(""),
		prevSlug: z.string().default(""),
		nextTitle: z.string().default(""),
		nextSlug: z.string().default(""),
    pinned: z.boolean().optional().default(false),
	}),
});
const specCollection = defineCollection({
	schema: z.object({}),
});
export const collections = {
	posts: postsCollection,
	spec: specCollection,
};
~~~
**2. src/utils/content-utils.ts**  
主要变化：在 getRawSortedPosts 中添加了优先判断 pinned 字段的逻辑  
~~~typescript
import { type CollectionEntry, getCollection } from "astro:content";
import I18nKey from "@i18n/i18nKey";
import { i18n } from "@i18n/translation";
import { getCategoryUrl } from "@utils/url-utils.ts";

// // Retrieve posts and sort them by publication date
async function getRawSortedPosts() {
	const allBlogPosts = await getCollection("posts", ({ data }) => {
		return import.meta.env.PROD ? data.draft !== true : true;
	});

	const sorted = allBlogPosts.sort((a, b) => {
		// --- 修改开始：置顶逻辑 ---
		const pinA = a.data.pinned ?? false;
		const pinB = b.data.pinned ?? false;

		if (pinA !== pinB) {
			return pinA ? -1 : 1; // 置顶的排在前面
		}
		// --- 修改结束 ---

		const dateA = new Date(a.data.published);
		const dateB = new Date(b.data.published);
		return dateA > dateB ? -1 : 1;
	});
	return sorted;
}

export async function getSortedPosts() {
	const sorted = await getRawSortedPosts();

	for (let i = 1; i < sorted.length; i++) {
		sorted[i].data.nextSlug = sorted[i - 1].slug;
		sorted[i].data.nextTitle = sorted[i - 1].data.title;
	}
	for (let i = 0; i < sorted.length - 1; i++) {
		sorted[i].data.prevSlug = sorted[i + 1].slug;
		sorted[i].data.prevTitle = sorted[i + 1].data.title;
	}

	return sorted;
}
export type PostForList = {
	slug: string;
	data: CollectionEntry<"posts">["data"];
};
export async function getSortedPostsList(): Promise<PostForList[]> {
	const sortedFullPosts = await getRawSortedPosts();

	// delete post.body
	const sortedPostsList = sortedFullPosts.map((post) => ({
		slug: post.slug,
		data: post.data,
	}));

	return sortedPostsList;
}
export type Tag = {
	name: string;
	count: number;
};

export async function getTagList(): Promise<Tag[]> {
	const allBlogPosts = await getCollection<"posts">("posts", ({ data }) => {
		return import.meta.env.PROD ? data.draft !== true : true;
	});

	const countMap: { [key: string]: number } = {};
	allBlogPosts.forEach((post: { data: { tags: string[] } }) => {
		post.data.tags.forEach((tag: string) => {
			if (!countMap[tag]) countMap[tag] = 0;
			countMap[tag]++;
		});
	});

	// sort tags
	const keys: string[] = Object.keys(countMap).sort((a, b) => {
		return a.toLowerCase().localeCompare(b.toLowerCase());
	});

	return keys.map((key) => ({ name: key, count: countMap[key] }));
}

export type Category = {
	name: string;
	count: number;
	url: string;
};

export async function getCategoryList(): Promise<Category[]> {
	const allBlogPosts = await getCollection<"posts">("posts", ({ data }) => {
		return import.meta.env.PROD ? data.draft !== true : true;
	});
	const count: { [key: string]: number } = {};
	allBlogPosts.forEach((post: { data: { category: string | null } }) => {
		if (!post.data.category) {
			const ucKey = i18n(I18nKey.uncategorized);
			count[ucKey] = count[ucKey] ? count[ucKey] + 1 : 1;
			return;
		}

		const categoryName =
			typeof post.data.category === "string"
				? post.data.category.trim()
				: String(post.data.category).trim();

		count[categoryName] = count[categoryName] ? count[categoryName] + 1 : 1;
	});

	const lst = Object.keys(count).sort((a, b) => {
		return a.toLowerCase().localeCompare(b.toLowerCase());
	});

	const ret: Category[] = [];
	for (const c of lst) {
		ret.push({
			name: c,
			count: count[c],
			url: getCategoryUrl(c),
		});
	}
	return ret;
}
~~~
**3. src/components/PostCard.astro**  
主要变化：在标题区域添加了图钉图标 (material-symbols:push-pin) 的显示逻辑
~~~typescript
---
import type { CollectionEntry } from "astro:content";
import path from "node:path";
import { Icon } from "astro-icon/components";
import I18nKey from "../i18n/i18nKey";
import { i18n } from "../i18n/translation";
import { getDir } from "../utils/url-utils";
import ImageWrapper from "./misc/ImageWrapper.astro";
import PostMetadata from "./PostMeta.astro";

interface Props {
	class?: string;
	entry: CollectionEntry<"posts">;
	title: string;
	url: string;
	published: Date;
	updated?: Date;
	tags: string[];
	category: string | null;
	image: string;
	description: string;
	draft: boolean;
	style: string;
}
const {
	entry,
	title,
	url,
	published,
	updated,
	tags,
	category,
	image,
	description,
	style,
} = Astro.props;
const className = Astro.props.class;

const hasCover = image !== undefined && image !== null && image !== "";

const coverWidth = "28%";

const { remarkPluginFrontmatter } = await entry.render();
---
<div class:list={["card-base flex flex-col-reverse md:flex-col w-full rounded-[var(--radius-large)] overflow-hidden relative", className]} style={style}>
    <div class:list={["pl-6 md:pl-9 pr-6 md:pr-2 pt-6 md:pt-7 pb-6 relative", {"w-full md:w-[calc(100%_-_52px_-_12px)]": !hasCover, "w-full md:w-[calc(100%_-_var(--coverWidth)_-_12px)]": hasCover}]}>
        <a href={url}
           class="transition group w-full block font-bold mb-3 text-3xl text-90
        hover:text-[var(--primary)] dark:hover:text-[var(--primary)]
        active:text-[var(--title-active)] dark:active:text-[var(--title-active)]
        before:w-1 before:h-5 before:rounded-md before:bg-[var(--primary)]
        before:absolute before:top-[35px] before:left-[18px] before:hidden md:before:block
        ">
            {/* --- 修改开始：置顶图标 --- */}
            {entry.data.pinned && (
                <Icon 
                    name="material-symbols:push-pin" 
                    class="inline-block mr-2 text-[var(--primary)] align-middle text-[1.6rem] mb-1 rotate-45"
                ></Icon>
            )}
            {/* --- 修改结束 --- */}
            
            {title}
            <Icon class="inline text-[2rem] text-[var(--primary)] md:hidden translate-y-0.5 absolute" name="material-symbols:chevron-right-rounded" ></Icon>
            <Icon class="text-[var(--primary)] text-[2rem] transition hidden md:inline absolute translate-y-0.5 opacity-0 group-hover:opacity-100 -translate-x-1 group-hover:translate-x-0" name="material-symbols:chevron-right-rounded"></Icon>
        </a>

        <!-- metadata -->
        <PostMetadata published={published} updated={updated} tags={tags} category={category} hideTagsForMobile={true} hideUpdateDate={true} class="mb-4"></PostMetadata>

        <!-- description -->
        <div class:list={["transition text-75 mb-3.5 pr-4", {"line-clamp-2 md:line-clamp-1": !description}]}>
            { description || remarkPluginFrontmatter.excerpt }
        </div>

        <!-- word count and read time  -->
        <div class="text-sm text-black/30 dark:text-white/30 flex gap-4 transition">
            <div>
                {remarkPluginFrontmatter.words} {" " + i18n(remarkPluginFrontmatter.words === 1 ? I18nKey.wordCount : I18nKey.wordsCount)}
            </div>
            <div>|</div>
            <div>
                {remarkPluginFrontmatter.minutes} {" " + i18n(remarkPluginFrontmatter.minutes === 1 ? I18nKey.minuteCount : I18nKey.minutesCount)}
            </div>
        </div>

    </div>

    {hasCover && <a href={url} aria-label={title}
                    class:list={["group",
                        "max-h-[20vh] md:max-h-none mx-4 mt-4 -mb-2 md:mb-0 md:mx-0 md:mt-0",
                        "md:w-[var(--coverWidth)] relative md:absolute md:top-3 md:bottom-3 md:right-3 rounded-xl overflow-hidden active:scale-95"
                    ]} >
        <div class="absolute pointer-events-none z-10 w-full h-full group-hover:bg-black/30 group-active:bg-black/50 transition"></div>
        <div class="absolute pointer-events-none z-20 w-full h-full flex items-center justify-center ">
            <Icon name="material-symbols:chevron-right-rounded"
                  class="transition opacity-0 group-hover:opacity-100 scale-50 group-hover:scale-100 text-white text-5xl">
            </Icon>
        </div>
        <ImageWrapper src={image} basePath={path.join("content/posts/", getDir(entry.id))} alt="Cover Image of the Post"
                  class="w-full h-full">
        </ImageWrapper>
    </a>}

    {!hasCover &&
        <a href={url} aria-label={title} class="!hidden md:!flex btn-regular w-[3.25rem]
         absolute right-3 top-3 bottom-3 rounded-xl bg-[var(--enter-btn-bg)]
         hover:bg-[var(--enter-btn-bg-hover)] active:bg-[var(--enter-btn-bg-active)] active:scale-95
        ">
            <Icon name="material-symbols:chevron-right-rounded"
                  class="transition text-[var(--primary)] text-4xl mx-auto">
            </Icon>
        </a>
    }
</div>
<div class="transition border-t-[1px] border-dashed mx-6 border-black/10 dark:border-white/[0.15] last:border-t-0 md:hidden"></div>

<style define:vars={{coverWidth}}>
</style>
~~~
