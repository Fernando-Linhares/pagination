<script setup>
    import { defineProps, onMounted, reactive, watch } from 'vue';

    const props = defineProps({
        pages: Array,
        paginate: Function,
        pagesTotal: Function,
        amount: Function
    });

    const data = reactive({
        pages: [],
        currentPage: 1
    });


    watch(
        () => props.pages.length,
        () => loadPagesAsync()
    );

    onMounted(async () => await loadPagesAsync());

    const loadPagesAsync = () => new Promise((resolve, reject) =>{
        try {
            setTimeout(() => {
                loadPages();
                resolve();
            }, 100)
        } catch (error) {
            reject(error);
        }
    });

    const loadPages = () => {

        let initalPages = [];

        let dynmaicPages = props.pages;

        if(props.pages.length < 7) {

            initalPages = dynmaicPages
            .slice(0, 7)
            .map(p => {
                return {
                    id: p.content,
                    content: p.content,
                    selected: p.content == 1
                }
            });

            definePages(initalPages);

            return;
        }

        initalPages = dynmaicPages
            .slice(0, 7)
            .map(p => {
                return {
                    id: p.content,
                    content: p.content,
                    selected: p.content == 1
                }
            });

        initalPages[initalPages.length - 1].content = props.pages.length;
        initalPages[initalPages.length - 2].content = '...';

        definePages(initalPages);
    }

    const selectPage = page => {

        data.currentPage = page.id;

        if(page.content == '...' && page.id == 1)
            return before();

        if(page.content == '...' && page.id == (data.pages.length - 1))
            return next();

        data.pages.forEach(page => page.selected = false);

        page.selected = true;

        let start = props.amount() * (page.content - 1);

        let end = props.amount() + start;

        props.paginate(start, end);

        if(props.pages.length > 7  && data.pages[0].content == '...' && page.id == 2)
            normalizePagesDecrement(page);

        if((props.pages.length > 7 && page.id == 5) && page.content + 2 !== props.pages.length)
            normalizePages(page);
    };

    const before = () => {
        selectPage(data.pages[1]);
    };

    const next = () => {
        selectPage(data.pages[4]);
    };

    const normalizePages = page => {

        let initalPages = [];

        let dynmaicPages = props.pages;

        let root = page.content - 5;

        let start = root + 1;

        let end = start + 7;

        let id = 0;

        initalPages = dynmaicPages
            .slice(start, end)
            .map(p => {
                id++;
                return {
                    id: id,
                    content: p.content,
                    selected: id == 4
                }
            });

        initalPages[0].content = '...';

        if(page.content + 3 !== props.pages.length) {
            initalPages[initalPages.length - 1].content = props.pages.length;
            initalPages[initalPages.length - 2].content = '...';
        }

        redefinePages(initalPages);
    };

    const normalizePagesDecrement = page => {

        let initalPages = [];

        let dynmaicPages = props.pages;

        let root = page.content - 2;

        let start =  root - 1;

        let end = start + 7;

        if(root == 0) {
            let id = 0;

            initalPages = dynmaicPages
            .slice(0, 7)
            .map(p => {
                id++;
                return {
                    id: id,
                    content: p.content,
                    selected: id == 2
                }
            });

            if(page.content + 3 !== props.pages.length) {
                initalPages[initalPages.length - 1].content = props.pages.length;
                initalPages[initalPages.length - 2].content = '...';
            }

            redefinePages(initalPages);

            return;
        }

        let id = 0;

        initalPages = dynmaicPages
            .slice(start, end)
            .map(p => {
                id++;
                return {
                    id: id,
                    content: p.content,
                    selected: id == 3
                }
            });


        if(page.content - 1 !== 0)
            initalPages[0].content = '...';

        if(page.content + 3 !== props.pages.length) {
            initalPages[initalPages.length - 1].content = props.pages.length;
            initalPages[initalPages.length - 2].content = '...';
        }

        redefinePages(initalPages);
    };

    const definePages = (pages) => data.pages = pages;

    const redefinePages = pages => {
        let i = 0;

        while(i < 7) {
            data.pages[i].content = pages[i].content;
            data.pages[i].selected =  pages[i].selected;
            i++;
        }
    }

    const selected = () => data.pages.filter(p => p.selected)[0];

    const lastPage = event => {

        if(!event.target.classList.contains('disabled')) {
        
            let page = selected();

            let npage = data.pages.filter(p => p.id == page.id - 1)[0];

            selectPage(npage);
        }
    };

    const nextPage = event => {

        if(!event.target.classList.contains('disabled')) {

            let page = selected();

            let npage = data.pages.filter(p => p.id == page.id + 1)[0];

            selectPage(npage);
        }
    }

</script>

<template>
     <nav aria-label="Page navigation example">
        <ul class="pagination justify-content-center" >
            <li class="page-item disabled" v-if="data.currentPage == 1">
                <span class="page-link" @click="lastPage($event)" tabindex="-1" aria-disabled="true">Previous</span>
            </li>
            <li class="page-item" v-else>
                <span class="page-link" @click="lastPage($event)" tabindex="-1" aria-disabled="true">Previous</span>
            </li>
            <li v-for="page in data.pages" :key="page.id" class="page-item" >
                <span :class="page.selected ? 'page-link active' : 'page-link' " @click="selectPage(page)" >{{ page.content }}</span>
            </li>

            <li class="page-item disabled" v-if="data.currentPage == data.pages.length">
                <span class="page-link" @click="nextPage($event)" tabindex="-1" aria-disabled="true">Next</span>
            </li>
            <li class="page-item" v-else>
                <span class="page-link" @click="nextPage($event)" tabindex="-1" aria-disabled="true">Next</span>
            </li>
        </ul>
    </nav>
</template>