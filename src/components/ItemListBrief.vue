<template>
    <div class="item-list-brief">
        <p class="headline">
            <span class="date date-span" v-if="fields.date && fields.end">{{ fields.date }} - {{ fields.end }}</span>
            <span class="date date-single" v-else-if="fields.date">{{ fields.date }}</span>
            <g-link class="title" :to="fields.link" :target="target">{{ fields.title }}</g-link>
        </p>
        <p class="tease markdown" v-if="fields.tease" v-html="mdToHtml(fields.tease)"></p>
    </div>
</template>

<script>
import { mdToHtml } from "~/lib/utils.js";
export default {
    props: {
        // Can be an Article, a VueArticle, or any object with a `title`, `link`, and (optionally) a `tease` field.
        item: { type: Object, required: true },
        target: { type: String, required: false, default: "" },
    },
    methods: {
        mdToHtml,
    },
    computed: {
        fields() {
            /** Normalize the input object into a standard set of fields.
             *  This will parse the metadata from an Article into the standard set of "item" fields.
             */
            let fields = {
                date: this.item.date,
                end: this.item.end !== this.item.date && this.item.end,
                title: this.item.title,
                link: this.item.link || this.item.external_url || this.item.path,
                tease: this.item.tease || "",
            };
            if (this.item.location) {
                fields.tease += ` ${this.item.location}`;
            }
            if (this.item.closes) {
                // The date a job opening closes.
                fields.tease += `. **Apply by ${this.item.closes}**`;
            }
            return fields;
        },
    },
};
</script>

<style scoped>
.item-list-brief {
    padding: 8px 12px;
    margin: 0;
}
.headline {
    margin: 0;
}
.date {
    display: inline-block;
    font-weight: bold;
}
.date-single {
    width: 3.3rem;
}
.date-span {
    margin-right: 0.5rem;
}
.title {
    font-weight: bold;
}
.tease {
    font-size: 80%;
    margin-top: 2px;
    margin-bottom: 0;
}
</style>
