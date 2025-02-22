<template>
    <Layout :subsite="subsite">
        <header id="header">
            <h1 class="title">{{ title }}</h1>
            <h3 v-if="subtitle">{{ subtitle }}</h3>
        </header>

        <HomeTop :lead="bundles.lead" :jumbotron="inserts.jumbotron" />

        <Bundle id="main-content" class="row" :bundle="bundles.main" :subclasses="['col-sm-12']" />

        <div class="row" v-for="(cardRow, i) of cardRows" :key="i">
            <HomeCard
                v-for="(card, j) of cardRow"
                :key="j"
                :title="card.title"
                :link="card.link"
                :icon="card.icon"
                :width="card.width"
                :items="card.items"
                :content="card.content"
            />
        </div>

        <Bundle class="lower" :bundle="bundles.lower" />
    </Layout>
</template>

<script>
import HomeTop from "@/components/HomeTop";
import HomeCard from "@/components/HomeCard";
import Bundle from "@/components/Bundle";
import {
    gatherInserts,
    gatherCollections,
    gatherBundles,
    searchBundle,
    gatherCards,
    makeCardRows,
} from "~/lib/pages.mjs";
import { addTwitterScript, addAltmetricsScript } from "~/lib/client.mjs";
import CONFIG from "~/../config.json";
export default {
    components: {
        HomeTop,
        HomeCard,
        Bundle,
    },
    metaInfo() {
        return {
            title: this.inserts.main ? this.inserts.main.title : this.subsiteData.name,
        };
    },
    created() {
        this.subsite = this.$context.subsite;
        this.subsiteData = CONFIG.subsites.all[this.$context.subsite];
        this.inserts = gatherInserts(this.$page.allInsert);
        this.cards = gatherCards(this.inserts);
        this.latest = gatherCollections(this.$page);
        this.bundles = gatherBundles(this.inserts);
    },
    computed: {
        title() {
            return searchBundle(this.bundles.main, "title", this.subsiteData.name);
        },
        subtitle() {
            return searchBundle(this.bundles.main, "subtitle");
        },
        cardRows() {
            return makeCardRows(this.$page.cards, this.latest, this.cards, `/${this.$context.subsite}`);
        },
    },
    mounted() {
        // Insert Twitter feed.
        if (this.cards.twitter) {
            addTwitterScript(window);
        }
        // Add altmetrics stats badges to publications.
        if (this.cards.pubs) {
            addAltmetricsScript(window);
        }
    },
};
</script>

<page-query>
query($subsite: String, $cardsPath: String, $insertRegex: String) {
    cards: insert(path: $cardsPath) {
        id
        list {
            name
            type
            title
            link
            icon
            width
        }
    }
    allInsert(filter: {path: {regex: $insertRegex}}) {
        totalCount
        edges {
            node {
                id
                path
                title
                subtitle
                content
                link
                icon
                items {
                    title
                    link
                    tease
                }
                fileInfo {
                    path
                }
            }
        }
    }
    news: allArticle(
        limit: 5, filter: {category: {eq: "news" }, subsites: {contains: [$subsite]}, draft: {ne: true}}
    ) {
        totalCount
        edges {
            node {
                ...articleFields
            }
        }
    }
    events: allArticle(
        limit: 5, sortBy: "date", order: ASC,
        filter: {
            category: {eq: "events"}, subsites: {contains: [$subsite]}, has_date: {eq: true}, days_ago: {lte: 0},
            draft: {ne: true}
        }
    ) {
        totalCount
        edges {
            node {
                ...articleFields
                date (format: "MMM D")
                end  (format: "MMM D")
            }
        }
    }
}
fragment articleFields on Article {
    id
    title
    tease
    external_url
    path
}
</page-query>

<style scoped>
#header {
    margin-bottom: 2.5rem;
}
#header .title {
    font-size: 3rem;
}
#main-content {
    margin-bottom: 20px;
}
</style>
