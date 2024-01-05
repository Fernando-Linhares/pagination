<script setup>
    import { onMounted, reactive } from 'vue';

    const data = reactive({
        collumns: Array,
        rows: Array,
        amount: Array,
        page: Number,
        dynamicRows: Array,
        order: Object,
        pages: Array,
        pages_btn: Array
    });

    data.collumns = [
        'id',
        'name',
        'price'
    ];

    data.rows = [];

    for(let i = 0; i < 200; i++) {
        data.rows.push({
            id: i+1,
            name: 'wsas'+ String(i *2 / 3),
            price: i + 6 +12.23
        });
    }

    data.amount = [
        10,
        50,
        100
    ];

    const actions = {
        permission: { create: true, update: true, delete: true },
        buttons: {
            create: {
                text: '',
                class: 'btn btn-primary float-end',
                icon: 'add_box',
                click: () => alert('ola mundo')
            },
            update: {
                text: '',
                class: 'btn btn-success',
                icon: 'mode_edit',
                click: () => alert('ola mundo')
            },
            delete: {
                text: '',
                class: 'btn btn-danger',
                icon: 'delete',
                click: () => alert('ola mundo')
            }
        }
    }

    const loadAmount = (event) => {

        let page = data.pages.filter(p => p.selected)[0];

        let amount = data.amount[event.target.selectedIndex];

        let currentPage = amount * (page.content - 1);

        let start = currentPage;

        let end = amount + currentPage;

        paginate(start, end);
    };

    onMounted(() => {

        data.pages = [];

        for(let i = 0; i < pagesTotal(); i++) {
    
            if(i == 0) {
                data.pages.push({content: i + 1, selected: true});
                continue;
            }

            if(i > 5)
                break;
    
            if(i > 3) {
                data.pages.push({content:'.', selected: false});
                continue;
            }

            data.pages.push({content:i + 1, selected: false});
        }


        paginate(0, 10);
    });

    data.order = {
        columnOrder: 'id',
        asc: true
    };

    const orderBy = column => {

        if(column !== data.order.columnOrder)
            data.order.columnOrder = column;

        data.order.asc = !data.order.asc;

        data.dynamicRows = data
        .rows
        .sort(() => -1)
        .slice(data.page, data.amount[document.querySelector('.form-select').selectedIndex]);
    }

    const pagesTotal = () => data.rows.length / data.amount[document.querySelector('.form-select').selectedIndex];

    const selectPage = page => {

        data.pages.forEach(page => page.selected = false);

        page.selected = true

        let amount = data.amount[document.querySelector('.form-select').selectedIndex];

        let currentPage = amount * (page.content - 1);

        let start = currentPage;

        let end = amount + currentPage;

        paginate(start, end);

        resolveOthers(page)
    };

    const resolveOthers = (currentPage) => {

        for(let i = (currentPage.content - 1); i < pagesTotal(); i++) {

            if(i == (currentPage.content - 1)) {
                data.pages.push({content: '.', selected: true});
                continue;
            }

            if(i > (currentPage.content + 5)) break;

            if(i > (currentPage.content + 3)) {
                data.pages.push({content:'.', selected: false});
                continue;
            }

            data.pages.push({content:i, selected: false});
        }
    };

    const paginate = (start, end) => data.dynamicRows = data
        .rows
        .slice(start, end);

</script>

<template>
        
    <div class="container">
        <div class="card card-bordered">
            <div class="card-title">
                <h3 class="fw-bolder p-3">
                    Pagination Title
                </h3>
            </div>
            <div class="card-body">
                <button v-show="actions.permission.create" :class="actions.buttons.create.class" @click="actions.buttons.create.click">
                    {{ actions.buttons.create.text }}
                    <i class="material-icons">
                        {{ actions.buttons.create.icon }}
                    </i>
                </button>
                    
                <div class="row">
                    <div class="col-md-2 col-lg-2 col-sm-12" >
                        <select class="form-select" aria-label="Pagination" @click="loadAmount($event)">
                            <option v-for="a in [10, 50, 100]" :key="a" :value="a"> {{ a }} </option>
                        </select>
                    </div>
                </div>

                <table class="table table-striped border">
                    <thead>
                        <tr>
                            <th :key="column" v-for="column in data.collumns" @click="orderBy(column)">
                                <span>
                                    {{ column }}
                                </span>
                                <i class="material-icons">
                                    sort
                                </i>
                            </th>
                            <th v-show="actions" style="width: 15%;">
                                actions
                            </th>
                        </tr>
                    </thead>
                    <tbody>
                        <tr v-for="row in data.dynamicRows" :key="row.id" >
                            <td v-for="column in data.collumns" :key="column.id">
                                {{ row[column] }}
                            </td>
                            <td v-show="actions">
                                <button
                                    v-show="actions.permission.update"
                                    :class="actions.buttons.update.class"
                                    @click="actions.buttons.update.click"
                                >
                                    {{ actions.buttons.update.text }}
                                    <i class="material-icons" v-show="actions.buttons.update.icon">
                                        {{ actions.buttons.update.icon }}
                                    </i>
                                </button>
                                &nbsp;
                                <button
                                    v-show="actions.permission.delete"
                                    :class="actions.buttons.delete.class"
                                    @click="actions.buttons.delete.click"

                                >
                                    {{ actions.buttons.delete.text }}
                                    <i class="material-icons" v-show="actions.buttons.delete.icon">
                                        {{ actions.buttons.delete.icon }}
                                    </i>
                                </button>
                            </td>
                        </tr>
                    </tbody>
                </table>

                <nav aria-label="Page navigation example">
                    <ul class="pagination justify-content-center" >
                        <li class="page-item disabled">
                            <span class="page-link" tabindex="-1" aria-disabled="true">Previous</span>
                        </li>

                        <li v-for="page in data.pages_btn" :key="page.content" class="page-item" >
                            <span :class="page.selected ? 'page-link active' : 'page-link' " @click="selectPage(page)" >{{ page.content }}</span>
                        </li>
                        <li class="page-item">
                            <span class="page-link" >Next</span>
                        </li>
                    </ul>
                </nav>

            </div>
        </div>

    </div>
</template>

<style>
th > i.material-icons
{
    color: grey;
    margin-left: 20px;
}

.page-link
{
    cursor: pointer;
}

#btn-create
{
    text-align: left;
    margin: auto;
}
</style>