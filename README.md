```
Mỗi cột được định nghĩa thông qua một mảng các đối tượng columns. Mỗi đối tượng trong columns sẽ bao gồm thông tin về tiêu đề cột (header), khóa dữ liệu (key), và các tùy chọn khác nếu cần.

Cấu trúc dữ liệu động
ts
Sao chép mã
export interface Column {
    key: string;
    label: string;
    width?: string;
    align?: 'start' | 'center' | 'end';
}

export interface TableProps {
    columns: Column[];
    data: Record<string, any>[];
    loading: boolean;
}
2. Component Table Động
vue
Sao chép mã
<script setup lang="ts">
import { defineProps } from 'vue';

const props = defineProps<TableProps>();

const onRowClick = (row: Record<string, any>) => {
    console.log('Row clicked:', row);
    // Xử lý hành động khi người dùng click vào một hàng
};
</script>

<template>
    <v-card variant="outlined" class="rounded-8">
        <v-card-text>
            <v-data-table
                :headers="props.columns"
                :items="props.data"
                :loading="props.loading"
                item-value="id"
                class="elevation-1"
                hide-default-footer
                dense
            >
                <template v-slot:top>
                    <div class="d-flex justify-end">
                        <v-btn color="primary" @click="$emit('create-item')">
                            Add New
                        </v-btn>
                    </div>
                </template>

                <template v-slot:body="{ props: { items } }">
                    <tbody>
                        <tr v-for="(item, index) in items" :key="index" @click="onRowClick(item)">
                            <td v-for="column in props.columns" :key="column.key" :style="{ textAlign: column.align || 'start' }">
                                {{ item[column.key] }}
                            </td>
                        </tr>
                    </tbody>
                </template>
            </v-data-table>
        </v-card-text>
    </v-card>
</template>
3. Sử dụng bảng động
Tại nơi bạn muốn sử dụng bảng, chỉ cần truyền danh sách columns và data.

vue
Sao chép mã
<script setup lang="ts">
import DynamicTable from '@/components/DynamicTable.vue';

const columns = [
    { key: 'id', label: 'ID', width: '10%' },
    { key: 'name', label: 'Name', width: '30%' },
    { key: 'price', label: 'Price', width: '20%', align: 'end' },
    { key: 'type', label: 'Type', width: '20%' },
    { key: 'action', label: 'Action', width: '20%' },
];

const data = ref([
    { id: '1', name: 'Product 1', price: '$100', type: 'Electronics' },
    { id: '2', name: 'Product 2', price: '$200', type: 'Furniture' },
]);

const loading = ref(false);

const onCreateItem = () => {
    console.log('Add new item clicked');
};

const onRowClicked = (row: Record<string, any>) => {
    console.log('Row clicked:', row);
};
</script>

<template>
    <DynamicTable
        :columns="columns"
        :data="data"
        :loading="loading"
        @create-item="onCreateItem"
        @row-clicked="onRowClicked"
    />
</template>
4. Hỗ trợ các cột hành động
Nếu cần thêm các hành động vào từng dòng (ví dụ: nút Edit hoặc Delete), bạn có thể tùy chỉnh trong bảng:

Thêm Cột Hành Động
vue
Sao chép mã
<template v-slot:body="{ props: { items } }">
    <tbody>
        <tr v-for="(item, index) in items" :key="index" @click="onRowClick(item)">
            <td v-for="column in props.columns" :key="column.key" :style="{ textAlign: column.align || 'start' }">
                <template v-if="column.key === 'action'">
                    <v-btn icon color="primary" @click.stop="$emit('edit-item', item)">Edit</v-btn>
                    <v-btn icon color="error" @click.stop="$emit('delete-item', item)">Delete</v-btn>
                </template>
                <template v-else>
                    {{ item[column.key] }}
                </template>
            </td>
        </tr>
    </tbody>
</template>
```
```
1. Dynamic Field Handling
Thay vì định nghĩa các thuộc tính như id, name, price, hay type cứng nhắc, hãy chuyển sang sử dụng một cấu trúc linh hoạt, ví dụ như một mảng các đối tượng fields. Mỗi đối tượng trong mảng này sẽ mô tả một field cụ thể.

Cấu trúc động cho Controller
ts
Sao chép mã
export interface Condition {
    [key: string]: string | number | null | undefined;
}

export interface Field {
    key: string;
    label: string;
    placeholder?: string;
}

export interface Props {
    loading: boolean;
    condition: Condition;
    fields: Field[];
}

export class Controller {
    public $app;
    public props;
    public condition: Ref<Condition>;
    public fields: Field[];
    public templateRefs: Ref<Record<string, unknown>>;

    constructor(app: IApp, props: Props) {
        this.$app = app;
        this.props = props;
        this.fields = props.fields;
        this.condition = ref<Condition>({ ...this.props.condition });
        this.templateRefs = ref({});
    }

    public getFieldValue = (key: string): string | number | null | undefined => {
        return this.condition.value[key];
    };

    public setFieldValue = (key: string, value: string | number | null) => {
        this.condition.value[key] = value;
    };
}

export const useController = (app: IApp, props: Props) => new Controller(app, props);
Tạo Form Động
vue
Sao chép mã
<script setup lang="ts">
import { inject } from 'vue';
import MTextField from '@/components/molecules/MTextField/index.vue';
import { useController as ctrl, type Props, type Field } from './useController';

import { Key } from '@/consts';
const $app = inject(Key);
if (!$app) throw new Error('No app provided');

const emits = defineEmits<{
    (e: 'search', condition: Record<string, string | number | null | undefined>): void;
    (e: 'reset'): void;
}>();

const props = withDefaults(defineProps<Props>(), {
    loading: true,
    fields: [] as Field[],
});

const { condition, fields, setFieldValue, getFieldValue } = ctrl($app, props);

const onSearch = () => {
    emits('search', { ...condition.value });
};

const onReset = () => {
    Object.keys(condition.value).forEach((key) => {
        condition.value[key] = null;
    });
    emits('reset');
};
</script>

<template>
    <v-card variant="outlined" class="rounded-8">
        <v-card-text>
            <v-row dense v-for="field in fields" :key="field.key">
                <v-col cols="12" md="6">
                    <MTextField
                        v-model="condition[field.key]"
                        :label="field.label"
                        :placeholder="field.placeholder || field.label"
                        :hide-detail="true"
                    />
                </v-col>
            </v-row>
            <v-spacer />
            <v-row dense>
                <v-col cols="6">
                    <v-btn
                        rounded="xl"
                        class="w-100 gradient-border"
                        variant="flat"
                        :loading="props.loading"
                        @click="onSearch"
                    >
                        Search
                    </v-btn>
                </v-col>
                <v-col cols="6">
                    <v-btn rounded="xl" class="w-100 gradient-border" variant="flat" @click="onReset">
                        Reset
                    </v-btn>
                </v-col>
            </v-row>
        </v-card-text>
    </v-card>
</template>
Gửi Field Dữ Liệu
Khi sử dụng, chỉ cần truyền các trường cần thiết qua props:

vue
Sao chép mã
<script setup lang="ts">
import SearchForm from '@/components/SearchForm.vue';

const fields = [
    { key: 'id', label: 'ID', placeholder: 'Enter ID' },
    { key: 'name', label: 'Name', placeholder: 'Enter Name' },
    { key: 'price', label: 'Price', placeholder: 'Enter Price' },
    { key: 'type', label: 'Type', placeholder: 'Enter Type' },
];

const loading = ref(false);

const onSearch = (condition) => {
    console.log('Search with:', condition);
};

const onReset = () => {
    console.log('Reset');
};
</script>

<template>
    <SearchForm :fields="fields" :loading="loading" @search="onSearch" @reset="onReset" />
</template>
```
---------------------------------------------------------------
```
Bước 1: Chỉnh sửa index.vue để hiển thị bảng
index.vue:

vue
<script setup lang="ts">
import { inject, onMounted } from 'vue';
import { useController as ctrl, type Props } from './useController';
import DataTable from '@/components/DataTable.vue';  // Thành phần bảng động

import { Key } from '@/consts';
const $app = inject(Key);
if (!$app) throw new Error('No app provided');

const props = withDefaults(defineProps<Props>(), {
  loading: true,
  headers: [],
});

const emits = defineEmits<{
  (e: 'rowClicked', id?: string): void;
  (e: 'createRowClicked'): void;
}>();

const { loading, data, headers, load } = ctrl($app, props);

const onDetail = (id: string | undefined) => {
  emits('rowClicked', id);
};

const oncreateClicked = () => {
  emits('createRowClicked');
};

onMounted(async () => {
  try {
    await load();
  } catch (error: unknown) {
    console.error(error);
  }
});
</script>

<template>
  <h1 class="title">List Items</h1>
  <DataTable :loading="loading" :headers="headers" :data="data" @row-clicked="onDetail" />
  <div class="d-flex justify-end">
    <v-btn rounded="x1" class="mt-5 gradient-border customerBtn" :loading="loading" @click="oncreateClicked">Create Item</v-btn>
  </div>
  <v-pagination v-model="condition.page" :length="pageCount || 1" density="compact" @update-page="" />
</template>
Bước 2: Tạo thành phần DataTable.vueđể hiển thị bảng
DataTable.vue:

vue
<template>
  <v-card variant="outlined" class="rounded-8">
    <v-card-text>
      <v-data-table
        :headers="headers"
        :items="data"
        :loading="loading"
        @click:row="onRowClick"
      />
    </v-card-text>
  </v-card>
</template>

<script setup lang="ts">
import { defineProps, defineEmits } from 'vue';

const props = defineProps<{
  loading: boolean;
  headers: Array<{ text: string; value: string; sortable?: boolean }>;
  data: Array<Record<string, any>>;
}>();

const emits = defineEmits<{
  (e: 'row-clicked', id: string): void;
}>();

const onRowClick = (item: Record<string, any>) => {
  emits('row-clicked', item.id);
};
</script>
Bước 3: Chỉnh sửa useController.ts để quản lý dữ liệu bảng
useController.ts:

ts
import { type IApp } from "@/applications/application";

export interface Condition {
  page: number;
  [key: string]: any;
}

export interface Props {
  loading: boolean;
  headers: Array<{ text: string; value: string; sortable?: boolean }>;
}

export class Controller {
  public $app;
  public loading;
  public data;
  public headers;
  public condition;
  public pageCount;
  private readonly itemsPerPage = 5;

  constructor(app: IApp, props: Props) {
    this.$app = app;
    this.loading = ref(false);
    this.data = ref<Array<Record<string, any>>>([]);
    this.headers = ref(props.headers);
    this.condition = ref<Condition>({ page: 1 });
    this.pageCount = ref(0);
  }

  public load = async (): Promise<void> => {
    try {
      this.loading.value = true;
      await delay(500);
      const resp = await this.$app.repository.itemRepository.getMany(this.condition.value);
      if (resp.datas) {
        this.pageCount.value = Math.ceil(resp.count / this.itemsPerPage);
        this.data.value = resp.datas;
      }
    } finally {
      this.loading.value = false;
    }
  };

  public reload = async (): Promise<void> => {
    await this.load();
  };
}

export const useController = (app: IApp, props: Props) => {
  const controller = new Controller(app, props);
  return controller;
};
Bước 4: Sử dụng thành phần động với dữ liệu bảng
pages/product.vue:

vue
<template>
  <title>Products</title>
  <DataTable
    :headers="productHeaders"
    :data="data"
    :loading="loading"
    @row-clicked="onDetail"
  />
  <div class="d-flex justify-end">
    <v-btn rounded="x1" class="mt-5 gradient-border customerBtn" :loading="loading" @click="oncreateClicked">Create Product</v-btn>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue';
import { useController } from './useController';
import DataTable from '@/components/DataTable.vue';
import { Key } from '@/consts';
const $app = inject(Key);
if (!$app) throw new Error('No app provided');

const productHeaders = [
  { text: 'ID', value: 'id' },
  { text: 'Name', value: 'name' },
  { text: 'Price', value: 'price' },
  { text: 'Type', value: 'type' },
];

const { loading, data, setFields, load, headers } = useController($app, {
  loading: ref(false),
  headers: productHeaders,
});

setFields(productHeaders);

const onDetail = (id: string) => {
  console.log('Row clicked:', id);
  // Thực hiện chi tiết với ID
};

const oncreateClicked = () => {
  console.log('Create product clicked');
  // Thực hiện tạo mới sản phẩm
};

onMounted(async () => {
  try {
    await load();
  } catch (error: unknown) {
    console.error(error);
  }
});
</script>
```
```
Bước 1: Chỉnh sửa index.vue
Thay đổi index.vue để nhận và hiển thị các trường động:

index.vue:

vue
<script setup lang="ts">
import { inject } from 'vue';
import MTextField from '@/components/molecules/MTextField/index.vue';
import { useController as ctrl, type Props } from './useController';

import { Key } from '@/consts';
const $app = inject(Key);
if (!$app) throw new Error('No app provided');

const emits = defineEmits<{
  (e: 'search', values: Record<string, string | null>): void;
  (e: 'reset'): void;
}>();

const props = withDefaults(defineProps<Props>(), {
  loading: true,
  fields: []
});

const { condition, templateRefID, setFields } = ctrl($app, props);

const onSearch = async () => {
  emits('search', { ...condition.value });
};

const onReset = () => {
  Object.keys(condition.value).forEach(key => {
    condition.value[key] = '';
  });
  emits('reset');
};

setFields(props.fields);
</script>

<template>
  <v-card variant="outlined" class="rounded-8">
    <v-card-text>
      <v-row dense v-for="(field, index) in props.fields" :key="index">
        <v-col :cols="field.cols || 12" :md="field.md || 5">
          <MTextField
            v-model="condition[field.name]"
            :label="field.label"
            :placeholder="field.placeholder"
            :hide-detail="true"
            :auto-focus="field.autoFocus || false"
          />
        </v-col>
      </v-row>
      <v-spacer/>
      <v-row dense>
        <v-col cols="12" md="2">
          <v-row dense>
            <v-col cols="6" md="3">
              <v-btn rounded="xl" class="w-100 customerBtn gradient-border" variant="flat" :loading="props.loading" @click="onSearch">
                <span class="d-md-none">Search</span>
                <v-icon class="d-none d-md-block">mdi-magnify</v-icon>
              </v-btn>
            </v-col>
            <v-col cols="6" md="4">
              <v-btn rounded="xl" class="w-100 customerBtn gradient-border" variant="flat" @click="onReset">
                Reset
              </v-btn>
            </v-col>
          </v-row>
        </v-col>
      </v-row>
    </v-card-text>
  </v-card>
</template>
Bước 2: Chỉnh sửa useController.ts
Thay đổi useController.ts để quản lý các điều kiện động và các trường đầu vào:

useController.ts:

ts
import { type IApp } from "@/applications/application";

export interface Condition {
  [key: string]: string | null;
}

export interface Props {
  loading: boolean;
  fields: Field[];
}

export interface Field {
  name: string;
  label: string;
  placeholder: string;
  cols?: number;
  md?: number;
  autoFocus?: boolean;
}

export class Controller {
  public $app;
  public props;
  public condition: Ref<Condition>;
  public templateRefID: Ref<unknown>;

  constructor(app: IApp, props: Props) {
    this.$app = app;
    this.props = props;
    this.condition = ref<Condition>({});
    this.templateRefID = ref(null);
  }

  public setFields(fields: Field[]) {
    fields.forEach(field => {
      this.condition.value[field.name] = '';
    });
  }
}

export const useController = (app: IApp, props: Props) => {
  const controller = new Controller(app, props);
  return controller;
};
Bước 3: Sử dụng thành phần động
pages/product.vue:

vue
<template>
  <title>Products</title>
  <dynamic-form
    :fields="[
      { name: 'id', label: 'ID', placeholder: 'Input ID', autoFocus: true },
      { name: 'name', label: 'Name', placeholder: 'Input Name' },
      { name: 'price', label: 'Price', placeholder: 'Input Price' },
      { name: 'type', label: 'Type', placeholder: 'Input Type' }
    ]"
    :loading="loading"
    @search="onSearch"
    @reset="onReset"
  />
</template>

<script setup lang="ts">
import { ref } from 'vue';
import { useController } from './useController';
import DynamicForm from '@/components/DynamicForm.vue';
import { Key } from '@/consts';
const $app = inject(Key);
if (!$app) throw new Error('No app provided');

const { loading, setFields } = useController($app, {
  loading: ref(false),
  fields: [
    { name: 'id', label: 'ID', placeholder: 'Input ID', autoFocus: true },
    { name: 'name', label: 'Name', placeholder: 'Input Name' },
    { name: 'price', label: 'Price', placeholder: 'Input Price' },
    { name: 'type', label: 'Type', placeholder: 'Input Type' }
  ]
});

const onSearch = async (values: Record<string, string | null>) => {
  console.log('Search values:', values);
  // Thực hiện tìm kiếm với các giá trị đầu vào
};

const onReset = () => {
  console.log('Form reset');
};

</script>
```
```
Bước 1: Chỉnh sửa index.vue
Thay đổi index.vue để có thể nhận và hiển thị các trường động:

index.vue:

vue
<script setup lang="ts">
import { inject, onMounted } from 'vue';
import DynamicForm from '@/components/DynamicForm.vue';
import { useController as ctrl } from './useController';
import { Key } from '@/consts';

const $app = inject(Key);
if (!$app) throw new Error('No app provided');

const emits = defineEmits<{
  (e: 'rowClicked', id?: string): void;
  (e: 'createProductClicked'): void;
}>();

const { loading, pageCount, itemList, condition, load, fields } = ctrl($app);

const onDetail = (id: string | undefined) => {
  emits('rowClicked', id);
};

const onSearch = async (searchCondition: Record<string, string | null>) => {
  try {
    Object.assign(condition.value, searchCondition);
    condition.value.page = 1;
    await load();
  } catch (error: unknown) {
    console.error(error);
  }
};

const onReset = () => {
  Object.keys(condition.value).forEach(key => {
    condition.value[key] = undefined;
  });
  condition.value.page = 1;
  load();
};

const oncreateClicked = () => {
  emits('createProductClicked');
};

const onSort = async (key: string, order: string) => {
  condition.value.key = key;
  condition.value.order = order;
  await load();
};

const onPageClicked = async () => {
  await load();
};

onMounted(async () => {
  try {
    await load();
  } catch (error: unknown) {
    console.error(error);
  }
});
</script>

<template>
  <h1 class="title">List Items</h1>
  <DynamicForm :loading="loading" :fields="fields" @search="onSearch" @reset="onReset" />
  <div class="d-flex justify-end">
    <v-btn rounded="x1" class="mt-5 gradient-border customerBtn" :loading="loading" @click="oncreateClicked">Create Item</v-btn>
  </div>
  <ProductSearchResult :item-list="itemList" :loading="loading" :height="350" @on-detail-clicked="onDetail" />
  <v-pagination v-model="condition.page" :length="pageCount || 1" density="compact" @update-page="" />
</template>
Bước 2: Chỉnh sửa useController.ts
Thay đổi useController.ts để quản lý các điều kiện động và các trường đầu vào:

useController.ts:

ts
import { type IApp } from '@/applications/application';

export interface Condition {
  [key: string]: string | number | undefined;
}

export interface ItemData {
  [key: string]: string | number;
}

export interface Field {
  name: string;
  label: string;
  placeholder: string;
  cols?: number;
  md?: number;
  autoFocus?: boolean;
}

export class Controller {
  public $app;
  public loading;
  public condition;
  public itemList;
  public pageCount;
  public fields;
  private readonly itemsPerPage = 5;

  constructor(app: IApp) {
    this.$app = app;
    this.loading = ref(false);
    this.condition = ref<Condition>({});
    this.itemList = ref<ItemData[]>([]);
    this.pageCount = ref(0);
    this.fields = ref<Field[]>([]);
  }

  public setFields(fields: Field[]) {
    this.fields.value = fields;
    fields.forEach(field => {
      this.condition.value[field.name] = '';
    });
  }

  public load = async (): Promise<void> => {
    try {
      this.loading.value = true;
      await delay(500);
      const resp = await this.$app.repository.itemRepository.getMany(this.condition.value);
      if (resp.datas) {
        this.pageCount.value = Math.ceil(resp.count / this.itemsPerPage);
        this.itemList.value = resp.datas;
      }
    } finally {
      this.loading.value = false;
    }
  };

  public reload = async (): Promise<void> => {
    await this.load();
  };
}

export const useController = (app: IApp) => {
  const controller = new Controller(app);
  return controller;
};
Bước 3: Sử dụng DynamicForm và thiết lập các trường đầu vào
Bạn có thể sử dụng thành phần DynamicForm để thiết lập các trường đầu vào và quản lý chúng một cách động.

pages/product.vue:

vue
<template>
  <title>Products</title>
  <DynamicForm
    :fields="productFields"
    :loading="loading"
    @search="onSearch"
    @reset="onReset"
  />
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue';
import { useController } from './useController';
import DynamicForm from '@/components/DynamicForm.vue';
import { Key } from '@/consts';
const $app = inject(Key);
if (!$app) throw new Error('No app provided');

const { loading, setFields, load, fields } = useController($app);

const productFields = [
  { name: 'id', label: 'ID', placeholder: 'Input ID', autoFocus: true },
  { name: 'name', label: 'Name', placeholder: 'Input Name' },
  { name: 'price', label: 'Price', placeholder: 'Input Price' },
  { name: 'type', label: 'Type', placeholder: 'Input Type' },
];

setFields(productFields);

onMounted(async () => {
  try {
    await load();
  } catch (error: unknown) {
    console.error(error);
  }
});
</script>
pages/user.vue:

vue
<template>
  <title>Users</title>
  <DynamicForm
    :fields="userFields"
    :loading="loading"
    @search="onSearch"
    @reset="onReset"
  />
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue';
import { useController } from './useController';
import DynamicForm from '@/components/DynamicForm.vue';
import { Key } from '@/consts';
const $app = inject(Key);
if (!$app) throw new Error('No app provided');

const { loading, setFields, load, fields } = useController($app);

const userFields = [
  { name: 'username', label: 'Username', placeholder: 'Input Username', autoFocus: true },
  { name: 'address', label: 'Address', placeholder: 'Input Address' },
  { name: 'email', label: 'Email', placeholder: 'Input Email' },
];

setFields(userFields);

onMounted(async () => {
  try {
    await load();
  } catch (error: unknown) {
    console.error(error);
  }
});
</script>
```
```
1. Cập nhật Mock API (userapimock)
Thêm các hàm sau vào userapimock để xử lý thêm, xóa, sửa:

typescript
Sao chép mã
// Thêm mới user
async apiuseradd(newUser: { firstname: string; lastname: string; email: string }): Promise<{ message: string; user: any }> {
  const newId = this.userdatamock.length > 0 ? this.userdatamock[this.userdatamock.length - 1].id + 1 : 1;
  const user = { id: newId, ...newUser }; // Tạo user mới
  this.userdatamock.push(user); // Thêm vào mock data

  return {
    message: "User added successfully",
    user,
  };
}

// Xóa user
async apiuserdelete(userId: number): Promise<{ message: string }> {
  const index = this.userdatamock.findIndex((user) => user.id === userId);
  if (index === -1) {
    throw new Error("User not found");
  }
  this.userdatamock.splice(index, 1); // Xóa user
  return { message: "User deleted successfully" };
}

// Sửa user
async apiuserupdate(userId: number, updatedData: { firstname?: string; lastname?: string; email?: string }): Promise<{ message: string; user: any }> {
  const user = this.userdatamock.find((user) => user.id === userId);
  if (!user) {
    throw new Error("User not found");
  }
  Object.assign(user, updatedData); // Cập nhật user
  return {
    message: "User updated successfully",
    user,
  };
}
2. Cập nhật Repository (userrepository.ts)
Thêm các hàm tương ứng vào userrepository để kết nối với Mock API:

typescript
Sao chép mã
export class userrepository implements IUserReposity {
  private fetches: IFetches;

  constructor(fetches: IFetches) {
    this.fetches = fetches;
  }

  // Thêm user mới
  async add(newUser: { firstname: string; lastname: string; email: string }): Promise<{ message: string; user: any }> {
    const result = await this.fetches.user.apiuseradd(newUser);
    return result;
  }

  // Xóa user
  async delete(userId: number): Promise<{ message: string }> {
    const result = await this.fetches.user.apiuserdelete(userId);
    return result;
  }

  // Sửa user
  async update(userId: number, updatedData: { firstname?: string; lastname?: string; email?: string }): Promise<{ message: string; user: any }> {
    const result = await this.fetches.user.apiuserupdate(userId, updatedData);
    return result;
  }
}
3. Sử dụng trong useController.ts
Thêm các hàm để gọi repository và cập nhật giao diện:

typescript
Sao chép mã
// Thêm user
public addUser = async (newUser: { firstname: string; lastname: string; email: string }): Promise<void> => {
  this.loading.value = true;
  try {
    const response = await this.$app.repository.userrepository.add(newUser);
    console.log(response.message); // "User added successfully"
    this.load(); // Reload danh sách user sau khi thêm
  } catch (error) {
    console.error("Error adding user:", error.message);
  } finally {
    this.loading.value = false;
  }
};

// Xóa user
public deleteUser = async (userId: number): Promise<void> => {
  this.loading.value = true;
  try {
    const response = await this.$app.repository.userrepository.delete(userId);
    console.log(response.message); // "User deleted successfully"
    this.load(); // Reload danh sách user sau khi xóa
  } catch (error) {
    console.error("Error deleting user:", error.message);
  } finally {
    this.loading.value = false;
  }
};

// Sửa user
public updateUser = async (userId: number, updatedData: { firstname?: string; lastname?: string; email?: string }): Promise<void> => {
  this.loading.value = true;
  try {
    const response = await this.$app.repository.userrepository.update(userId, updatedData);
    console.log(response.message); // "User updated successfully"
    this.load(); // Reload danh sách user sau khi sửa
  } catch (error) {
    console.error("Error updating user:", error.message);
  } finally {
    this.loading.value = false;
  }
};
4. Cách sử dụng trong giao diện (component hoặc page)
Ví dụ: Thêm người dùng mới
typescript
Sao chép mã
await useController.addUser({
  firstname: "Alice",
  lastname: "Johnson",
  email: "alice.johnson@example.com",
});
Ví dụ: Xóa người dùng
typescript
Sao chép mã
await useController.deleteUser(2); // Xóa user với ID = 2
Ví dụ: Sửa người dùng
typescript
Sao chép mã
await useController.updateUser(1, {
  firstname: "John Updated",
  email: "john.updated@example.com",
});
Tóm tắt
Mock API (userapimock): Xử lý dữ liệu giả lập với các hàm thêm, xóa, sửa.
Repository (userrepository.ts): Giao tiếp với Mock API và trả về kết quả cho tầng controller.
Controller (useController.ts): Gọi repository và xử lý logic liên quan đến giao diện.
Với cấu trúc này, việc thêm, xóa, sửa dữ liệu người dùng sẽ được thực hiện xuyên suốt các tầng một cách rõ ràng và có tổ chức. Nếu bạn cần mở rộng thêm, hãy cho mình biết! 😊
```
```
// Giả lập dữ liệu mock ban đầu
this.userdatamock = [
  { id: 1, name: "John Doe", email: "john.doe@example.com" },
  { id: 2, name: "Jane Smith", email: "jane.smith@example.com" },
  { id: 3, name: "Bob Johnson", email: "bob.johnson@example.com" },
];

// Hàm lấy danh sách người dùng
async apiusergetmany(requestparameter: iapiusergetmanyrequest): Promise<iApiusergetmanyresponse> {
  let users = this.userdatamock; // Dữ liệu từ mock
  return {
    count: users.length, // Tổng số người dùng
    datas: users,        // Danh sách người dùng
  };
}

// Hàm thêm mới người dùng
async apiuseradd(newUser: { name: string; email: string }): Promise<{ message: string; user: any }> {
  // Tạo ID tự động cho người dùng mới
  const newId = this.userdatamock.length > 0 ? this.userdatamock[this.userdatamock.length - 1].id + 1 : 1;
  const user = { id: newId, ...newUser }; // Tạo đối tượng user mới
  
  this.userdatamock.push(user); // Thêm vào mảng dữ liệu mock

  return {
    message: "User added successfully",
    user,
  };
}

// Hàm xóa người dùng
async apiuserdelete(userId: number): Promise<{ message: string }> {
  // Tìm index của user cần xóa
  const index = this.userdatamock.findIndex((user) => user.id === userId);

  if (index === -1) {
    throw new Error("User not found"); // Nếu không tìm thấy
  }

  this.userdatamock.splice(index, 1); // Xóa user khỏi mảng
  return { message: "User deleted successfully" };
}

// Hàm sửa thông tin người dùng
async apiuserupdate(userId: number, updatedData: { name?: string; email?: string }): Promise<{ message: string; user: any }> {
  // Tìm user cần cập nhật
  const user = this.userdatamock.find((user) => user.id === userId);

  if (!user) {
    throw new Error("User not found"); // Nếu không tìm thấy
  }

  // Cập nhật thông tin
  Object.assign(user, updatedData);

  return {
    message: "User updated successfully",
    user,
  };
}

```
```
// --- Mock API ---
// File: userApiMock.js

let mockUserData = [
  { id: 1, firstname: 'John', lastname: 'Doe', email: 'john.doe@example.com' },
  { id: 2, firstname: 'Jane', lastname: 'Smith', email: 'jane.smith@example.com' },
  { id: 3, firstname: 'Alice', lastname: 'Johnson', email: 'alice.johnson@example.com' },
];

export default {
  async apiUserGetMany(searchCondition) {
    // Mock search functionality
    return new Promise((resolve) => {
      setTimeout(() => {
        const filteredData = mockUserData.filter((user) => {
          return (
            (!searchCondition.id || user.id === +searchCondition.id) &&
            (!searchCondition.firstname || user.firstname.includes(searchCondition.firstname)) &&
            (!searchCondition.lastname || user.lastname.includes(searchCondition.lastname)) &&
            (!searchCondition.email || user.email.includes(searchCondition.email))
          );
        });
        resolve({ count: filteredData.length, datas: filteredData });
      }, 500);
    });
  },

  async apiUserAdd(newUser) {
    return new Promise((resolve) => {
      setTimeout(() => {
        const newId = mockUserData.length ? mockUserData[mockUserData.length - 1].id + 1 : 1;
        const userToAdd = { id: newId, ...newUser };
        mockUserData.push(userToAdd);
        resolve(userToAdd);
      }, 500);
    });
  },

  async apiUserUpdate(userId, updatedUser) {
    return new Promise((resolve, reject) => {
      setTimeout(() => {
        const index = mockUserData.findIndex((user) => user.id === userId);
        if (index !== -1) {
          mockUserData[index] = { ...mockUserData[index], ...updatedUser };
          resolve(mockUserData[index]);
        } else {
          reject(new Error('User not found'));
        }
      }, 500);
    });
  },

  async apiUserDelete(userId) {
    return new Promise((resolve, reject) => {
      setTimeout(() => {
        const index = mockUserData.findIndex((user) => user.id === userId);
        if (index !== -1) {
          mockUserData.splice(index, 1);
          resolve({ success: true });
        } else {
          reject(new Error('User not found'));
        }
      }, 500);
    });
  },
};

// --- User Repository ---
// File: userRepository.js
import userApiMock from '@/mock/userApiMock';

export default {
  async getMany(searchCondition) {
    return userApiMock.apiUserGetMany(searchCondition);
  },

  async add(newUser) {
    return userApiMock.apiUserAdd(newUser);
  },

  async update(userId, updatedUser) {
    return userApiMock.apiUserUpdate(userId, updatedUser);
  },

  async delete(userId) {
    return userApiMock.apiUserDelete(userId);
  },
};

```

```
// --- Mock API ---
// File: userApiMock.js

let mockUserData = [
  { id: 1, firstname: 'John', lastname: 'Doe', email: 'john.doe@example.com' },
  { id: 2, firstname: 'Jane', lastname: 'Smith', email: 'jane.smith@example.com' },
  { id: 3, firstname: 'Alice', lastname: 'Johnson', email: 'alice.johnson@example.com' },
];

export default {
  async apiUserGetMany(searchCondition) {
    // Mock search functionality
    return new Promise((resolve) => {
      setTimeout(() => {
        const filteredData = mockUserData.filter((user) => {
          return (
            (!searchCondition.id || user.id === +searchCondition.id) &&
            (!searchCondition.firstname || user.firstname.includes(searchCondition.firstname)) &&
            (!searchCondition.lastname || user.lastname.includes(searchCondition.lastname)) &&
            (!searchCondition.email || user.email.includes(searchCondition.email))
          );
        });
        resolve({ count: filteredData.length, datas: filteredData });
      }, 500);
    });
  },

  async apiUserAdd(newUser) {
    return new Promise((resolve) => {
      setTimeout(() => {
        const newId = mockUserData.length ? mockUserData[mockUserData.length - 1].id + 1 : 1;
        const userToAdd = { id: newId, ...newUser };
        mockUserData.push(userToAdd);
        resolve(userToAdd);
      }, 500);
    });
  },

  async apiUserUpdate(userId, updatedUser) {
    return new Promise((resolve, reject) => {
      setTimeout(() => {
        const index = mockUserData.findIndex((user) => user.id === userId);
        if (index !== -1) {
          mockUserData[index] = { ...mockUserData[index], ...updatedUser };
          resolve(mockUserData[index]);
        } else {
          reject(new Error('User not found'));
        }
      }, 500);
    });
  },

  async apiUserDelete(userId) {
    return new Promise((resolve, reject) => {
      setTimeout(() => {
        const index = mockUserData.findIndex((user) => user.id === userId);
        if (index !== -1) {
          mockUserData.splice(index, 1);
          resolve({ success: true });
        } else {
          reject(new Error('User not found'));
        }
      }, 500);
    });
  },
};

// --- User Repository ---
// File: userRepository.js
import userApiMock from '@/mock/userApiMock';

export default {
  async getMany(searchCondition) {
    return userApiMock.apiUserGetMany(searchCondition);
  },

  async add(newUser) {
    return userApiMock.apiUserAdd(newUser);
  },

  async update(userId, updatedUser) {
    return userApiMock.apiUserUpdate(userId, updatedUser);
  },

  async delete(userId) {
    return userApiMock.apiUserDelete(userId);
  },
};

```
```
Cách tổ chức Mock API
Để quản lý Mock API tốt hơn, bạn có thể chia các phần sau trong thư mục Domain:

Repository Module:
Chứa các hàm thực hiện CRUD (Create, Read, Update, Delete). Đây là nơi bạn đặt logic API giả lập.

Handler (Middleware):
Xử lý nghiệp vụ cho các yêu cầu như kiểm tra tính hợp lệ của dữ liệu, tạo mã lỗi nếu cần.

Mock Data:
Chứa dữ liệu tĩnh ban đầu. Dữ liệu này sẽ được dùng trong quá trình phát triển.

Ví dụ cấu trúc thư mục
plaintext
Sao chép mã
domain/
├── repository/
│   └── userRepository.js     # Chứa các hàm CRUD cho User
├── handlers/
│   └── userHandler.js         # Xử lý logic nghiệp vụ trước khi gọi repository
├── mocks/
│   └── mockUsers.js           # Dữ liệu giả lập ban đầu
Triển khai cụ thể
1. mockUsers.js (Dữ liệu ban đầu)
javascript
Sao chép mã
export const mockUsers = [
  { id: 1, name: "John Doe", email: "john.doe@example.com" },
  { id: 2, name: "Jane Smith", email: "jane.smith@example.com" }
];
2. userRepository.js (Hàm CRUD)
javascript
Sao chép mã
import { mockUsers } from "../mocks/mockUsers";

// Hàm lấy danh sách user
export const fetchUsers = () => {
  return Promise.resolve(mockUsers);
};

// Hàm thêm mới user
export const addUser = (newUser) => {
  newUser.id = mockUsers.length + 1; // Tạo ID tự động
  mockUsers.push(newUser);
  return Promise.resolve(newUser);
};

// Hàm xóa user
export const deleteUser = (id) => {
  const index = mockUsers.findIndex((user) => user.id === id);
  if (index !== -1) {
    mockUsers.splice(index, 1);
    return Promise.resolve({ message: "User deleted successfully" });
  }
  return Promise.reject(new Error("User not found"));
};

// Hàm cập nhật user
export const updateUser = (id, updatedData) => {
  const user = mockUsers.find((user) => user.id === id);
  if (user) {
    Object.assign(user, updatedData);
    return Promise.resolve(user);
  }
  return Promise.reject(new Error("User not found"));
};
3. userHandler.js (Xử lý logic)
Bạn có thể thêm các kiểm tra hợp lệ hoặc xử lý lỗi tại đây trước khi gọi userRepository.

javascript
Sao chép mã
import { addUser, deleteUser, fetchUsers, updateUser } from "../repository/userRepository";

// Xử lý lấy danh sách user
export const handleFetchUsers = async () => {
  return await fetchUsers();
};

// Xử lý thêm mới user
export const handleAddUser = async (userData) => {
  if (!userData.name || !userData.email) {
    throw new Error("Name and Email are required");
  }
  return await addUser(userData);
};

// Xử lý xóa user
export const handleDeleteUser = async (id) => {
  if (!id) {
    throw new Error("User ID is required");
  }
  return await deleteUser(id);
};

// Xử lý cập nhật user
export const handleUpdateUser = async (id, updatedData) => {
  if (!id || !updatedData) {
    throw new Error("User ID and update data are required");
  }
  return await updateUser(id, updatedData);
};
Tích hợp với Frontend
Nên đặt API gọi ở đâu?
Nếu dùng trong nhiều nơi: Tạo một file composable/useUser.js trong thư mục composable để gọi các API từ userHandler.
Nếu chỉ dùng trong một trang cụ thể: Gọi trực tiếp từ component.
Composable/useUser.js
javascript
Sao chép mã
import {
  handleFetchUsers,
  handleAddUser,
  handleDeleteUser,
  handleUpdateUser
} from "@/domain/handlers/userHandler";

export const useUser = () => {
  const fetchUsers = async () => {
    try {
      return await handleFetchUsers();
    } catch (error) {
      console.error(error.message);
    }
  };

  const addUser = async (userData) => {
    try {
      return await handleAddUser(userData);
    } catch (error) {
      console.error(error.message);
    }
  };

  const deleteUser = async (id) => {
    try {
      return await handleDeleteUser(id);
    } catch (error) {
      console.error(error.message);
    }
  };

  const updateUser = async (id, updatedData) => {
    try {
      return await handleUpdateUser(id, updatedData);
    } catch (error) {
      console.error(error.message);
    }
  };

  return { fetchUsers, addUser, deleteUser, updateUser };
};
Trong Component (ví dụ: UserPage.vue)
vue
Sao chép mã
<template>
  <div>
    <h1>Users</h1>
    <ul>
      <li v-for="user in users" :key="user.id">
        {{ user.name }} ({{ user.email }})
        <button @click="removeUser(user.id)">Delete</button>
      </li>
    </ul>
    <form @submit.prevent="addNewUser">
      <input v-model="newUser.name" placeholder="Name" required />
      <input v-model="newUser.email" placeholder="Email" required />
      <button type="submit">Add User</button>
    </form>
  </div>
</template>

<script>
import { useUser } from "@/composable/useUser";

export default {
  data() {
    return {
      users: [],
      newUser: { name: "", email: "" },
    };
  },
  async created() {
    const { fetchUsers } = useUser();
    this.users = await fetchUsers();
  },
  methods: {
    async addNewUser() {
      const { addUser } = useUser();
      const user = await addUser(this.newUser);
      if (user) {
        this.users.push(user);
        this.newUser = { name: "", email: "" };
      }
    },
    async removeUser(id) {
      const { deleteUser } = useUser();
      await deleteUser(id);
      this.users = this.users.filter((user) => user.id !== id);
    },
  },
};
</script>
Tóm tắt nên đặt ở đâu
Logic xử lý API Mock (CRUD): Đặt trong repository.
Kiểm tra dữ liệu và xử lý lỗi: Đặt trong handler.
Gọi API từ frontend: Tạo file composable nếu dùng lại ở nhiều nơi.
Quản lý giao diện: Xử lý logic hiển thị trong component hoặc page.
Cấu trúc này giúp bạn dễ bảo trì và tái sử dụng code hiệu quả hơn. Nếu có điểm nào cần làm rõ, cứ hỏi thêm nhé! 😊
```
```
Cập nhật BlogList.vue với tính năng xóa bài viết
Cập nhật BlogList.vue để thêm nút xóa bài viết:

vue
Sao chép mã
<template>
  <div>
    <h1>Danh sách bài viết</h1>
    <div v-for="post in posts" :key="post.id" class="post">
      <h2>{{ post.title }}</h2>
      <p>{{ post.content.slice(0, 100) }}...</p>
      <router-link :to="`/post/${post.id}`">Xem thêm</router-link>
      <button @click="$emit('delete-post', post.id)">Xóa</button>
    </div>
  </div>
</template>

<script>
export default {
  props: ["posts"],
};
</script>

<style>
.post {
  border: 1px solid #ccc;
  padding: 10px;
  margin-bottom: 10px;
  display: flex;
  justify-content: space-between;
  align-items: center;
}
button {
  margin-left: 10px;
  color: white;
  background-color: red;
  border: none;
  padding: 5px 10px;
  cursor: pointer;
}
button:hover {
  background-color: darkred;
}
</style>
Cập nhật Home.vue để xử lý xóa bài viết
Thêm phương thức deletePost trong Home.vue:

vue
Sao chép mã
<template>
  <div>
    <BlogForm @add-post="addPost" />
    <BlogList :posts="posts" @delete-post="deletePost" />
  </div>
</template>

<script>
import BlogList from "../components/BlogList.vue";
import BlogForm from "../components/BlogForm.vue";

export default {
  components: { BlogList, BlogForm },
  data() {
    return {
      posts: [],
    };
  },
  methods: {
    addPost(post) {
      this.posts.push(post);
      this.savePosts();
    },
    deletePost(id) {
      this.posts = this.posts.filter((post) => post.id !== id);
      this.savePosts();
    },
    savePosts() {
      localStorage.setItem("posts", JSON.stringify(this.posts));
    },
    loadPosts() {
      const savedPosts = localStorage.getItem("posts");
      if (savedPosts) {
        this.posts = JSON.parse(savedPosts);
      }
    },
  },
  created() {
    this.loadPosts();
  },
};
</script>
Thêm tính năng chỉnh sửa bài viết
Cập nhật BlogForm.vue để hỗ trợ chỉnh sửa bài viết:

vue
Sao chép mã
<template>
  <div>
    <h2>{{ isEditing ? "Chỉnh sửa bài viết" : "Thêm bài viết mới" }}</h2>
    <form @submit.prevent="submitForm">
      <div>
        <label for="title">Tiêu đề:</label>
        <input v-model="title" id="title" required />
      </div>
      <div>
        <label for="content">Nội dung:</label>
        <textarea v-model="content" id="content" required></textarea>
      </div>
      <button type="submit">
        {{ isEditing ? "Cập nhật bài viết" : "Thêm bài viết" }}
      </button>
      <button v-if="isEditing" @click.prevent="cancelEdit">Hủy</button>
    </form>
  </div>
</template>

<script>
export default {
  props: ["editPost"],
  data() {
    return {
      title: "",
      content: "",
      isEditing: false,
      postId: null,
    };
  },
  watch: {
    editPost(newPost) {
      if (newPost) {
        this.title = newPost.title;
        this.content = newPost.content;
        this.isEditing = true;
        this.postId = newPost.id;
      }
    },
  },
  methods: {
    submitForm() {
      const post = {
        id: this.postId || Date.now(),
        title: this.title,
        content: this.content,
      };
      this.$emit("submit-post", post);
      this.resetForm();
    },
    cancelEdit() {
      this.resetForm();
    },
    resetForm() {
      this.title = "";
      this.content = "";
      this.isEditing = false;
      this.postId = null;
    },
  },
};
</script>
Cập nhật Home.vue để quản lý chỉnh sửa bài viết:

vue
Sao chép mã
<template>
  <div>
    <BlogForm @submit-post="handlePostSubmit" :edit-post="editPost" />
    <BlogList
      :posts="posts"
      @delete-post="deletePost"
      @edit-post="startEditPost"
    />
  </div>
</template>

<script>
import BlogList from "../components/BlogList.vue";
import BlogForm from "../components/BlogForm.vue";

export default {
  components: { BlogList, BlogForm },
  data() {
    return {
      posts: [],
      editPost: null,
    };
  },
  methods: {
    handlePostSubmit(post) {
      if (post.id) {
        // Update bài viết
        const index = this.posts.findIndex((p) => p.id === post.id);
        if (index !== -1) this.posts.splice(index, 1, post);
      } else {
        // Thêm bài viết mới
        this.posts.push(post);
      }
      this.savePosts();
    },
    startEditPost(postId) {
      this.editPost = this.posts.find((post) => post.id === postId);
    },
    deletePost(id) {
      this.posts = this.posts.filter((post) => post.id !== id);
      this.savePosts();
    },
    savePosts() {
      localStorage.setItem("posts", JSON.stringify(this.posts));
    },
    loadPosts() {
      const savedPosts = localStorage.getItem("posts");
      if (savedPosts) {
        this.posts = JSON.parse(savedPosts);
      }
    },
  },
  created() {
    this.loadPosts();
  },
};
</script>
Cải thiện giao diện với TailwindCSS
Cài đặt TailwindCSS:

bash
Sao chép mã
npm install -D tailwindcss postcss autoprefixer
npx tailwindcss init
Cấu hình tailwind.config.js:

javascript
Sao chép mã
module.exports = {
  content: ["./src/**/*.{vue,js,ts,jsx,tsx}"],
  theme: {
    extend: {},
  },
  plugins: [],
};
Thêm TailwindCSS vào src/assets/tailwind.css:

css
Sao chép mã
@tailwind base;
@tailwind components;
@tailwind utilities;
Cập nhật main.js để import TailwindCSS:

javascript
Sao chép mã
import "./assets/tailwind.css";
Bây giờ, bạn có thể sử dụng các class Tailwind để cải thiện giao diện.

Thêm hiệu ứng chuyển đổi trang
Sử dụng transition từ Vue.js:

App.vue:

vue
Sao chép mã
<template>
  <div id="app">
    <transition name="fade" mode="out-in">
      <router-view />
    </transition>
  </div>
</template>

<style>
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.5s;
}
.fade-enter,
.fade-leave-to {
  opacity: 0;
}
</style>
Triển khai blog
Để triển khai, bạn có thể dùng Netlify hoặc Vercel:

bash
Sao chép mã
npm run build
Upload thư mục dist lên dịch vụ triển khai.

Giờ đây, bạn đã có một blog đầy đủ chức năng với giao diện đẹp và dễ sử dụng! 🚀
```
```
1. Tạo dự án Vue.js
Trước tiên, bạn cần tạo dự án Vue.js mới:

bash
Sao chép mã
vue create vue-blog
cd vue-blog
npm run serve
2. Cấu trúc dự án
Tạo cấu trúc thư mục sau trong thư mục src:

css
Sao chép mã
src/
├── components/
│   ├── BlogList.vue
│   ├── BlogDetail.vue
│   ├── BlogForm.vue
├── views/
│   ├── Home.vue
│   ├── Post.vue
├── router/
│   └── index.js
├── App.vue
├── main.js
3. Cài đặt Vue Router
Cài đặt Vue Router để điều hướng giữa các trang:

bash
Sao chép mã
npm install vue-router
File src/router/index.js:

javascript
Sao chép mã
import Vue from "vue";
import VueRouter from "vue-router";
import Home from "../views/Home.vue";
import Post from "../views/Post.vue";

Vue.use(VueRouter);

const routes = [
  { path: "/", name: "Home", component: Home },
  { path: "/post/:id", name: "Post", component: Post },
];

const router = new VueRouter({
  mode: "history",
  routes,
});

export default router;
Cập nhật src/main.js:

javascript
Sao chép mã
import Vue from "vue";
import App from "./App.vue";
import router from "./router";

Vue.config.productionTip = false;

new Vue({
  router,
  render: (h) => h(App),
}).$mount("#app");
4. Component BlogList.vue
Danh sách các bài viết hiển thị trên trang chủ.

File src/components/BlogList.vue:

vue
Sao chép mã
<template>
  <div>
    <h1>Danh sách bài viết</h1>
    <div v-for="post in posts" :key="post.id" class="post">
      <h2>{{ post.title }}</h2>
      <p>{{ post.content.slice(0, 100) }}...</p>
      <router-link :to="`/post/${post.id}`">Xem thêm</router-link>
    </div>
  </div>
</template>

<script>
export default {
  props: ["posts"],
};
</script>

<style>
.post {
  border: 1px solid #ccc;
  padding: 10px;
  margin-bottom: 10px;
}
</style>
5. Component BlogDetail.vue
Hiển thị chi tiết bài viết.

File src/components/BlogDetail.vue:

vue
Sao chép mã
<template>
  <div v-if="post">
    <h1>{{ post.title }}</h1>
    <p>{{ post.content }}</p>
    <router-link to="/">Quay lại danh sách</router-link>
  </div>
</template>

<script>
export default {
  props: ["post"],
};
</script>
6. Component BlogForm.vue
Form thêm bài viết mới.

File src/components/BlogForm.vue:

vue
Sao chép mã
<template>
  <div>
    <h2>Thêm bài viết mới</h2>
    <form @submit.prevent="submitForm">
      <div>
        <label for="title">Tiêu đề:</label>
        <input v-model="title" id="title" required />
      </div>
      <div>
        <label for="content">Nội dung:</label>
        <textarea v-model="content" id="content" required></textarea>
      </div>
      <button type="submit">Thêm bài viết</button>
    </form>
  </div>
</template>

<script>
export default {
  data() {
    return {
      title: "",
      content: "",
    };
  },
  methods: {
    submitForm() {
      const newPost = {
        id: Date.now(),
        title: this.title,
        content: this.content,
      };
      this.$emit("add-post", newPost);
      this.title = "";
      this.content = "";
    },
  },
};
</script>
7. Trang Home.vue
Hiển thị danh sách bài viết và form thêm bài viết.

File src/views/Home.vue:

vue
Sao chép mã
<template>
  <div>
    <BlogForm @add-post="addPost" />
    <BlogList :posts="posts" />
  </div>
</template>

<script>
import BlogList from "../components/BlogList.vue";
import BlogForm from "../components/BlogForm.vue";

export default {
  components: { BlogList, BlogForm },
  data() {
    return {
      posts: [],
    };
  },
  methods: {
    addPost(post) {
      this.posts.push(post);
      this.savePosts();
    },
    savePosts() {
      localStorage.setItem("posts", JSON.stringify(this.posts));
    },
    loadPosts() {
      const savedPosts = localStorage.getItem("posts");
      if (savedPosts) {
        this.posts = JSON.parse(savedPosts);
      }
    },
  },
  created() {
    this.loadPosts();
  },
};
</script>
8. Trang Post.vue
Hiển thị chi tiết bài viết.

File src/views/Post.vue:

vue
Sao chép mã
<template>
  <BlogDetail :post="post" />
</template>

<script>
import BlogDetail from "../components/BlogDetail.vue";

export default {
  components: { BlogDetail },
  data() {
    return {
      post: null,
    };
  },
  created() {
    const posts = JSON.parse(localStorage.getItem("posts")) || [];
    const postId = Number(this.$route.params.id);
    this.post = posts.find((post) => post.id === postId);
  },
};
</script>
9. File App.vue
Tích hợp giao diện chính.

File src/App.vue:

vue
Sao chép mã
<template>
  <div id="app">
    <router-view />
  </div>
</template>

<script>
export default {
  name: "App",
};
</script>
10. Chạy ứng dụng
Khởi động ứng dụng:

bash
Sao chép mã
npm run serve
Truy cập vào http://localhost:8080 để xem blog của bạn.


```
```
Lịch học Vue.js (8 tiếng)
Giờ 1: Làm quen với Vue.js và thiết lập môi trường
Mục tiêu: Hiểu cơ bản về Vue.js và thiết lập môi trường làm việc.
Nội dung học:
Vue.js là gì? (Frontend Framework, SPA).
Tạo ứng dụng Vue.js đầu tiên.
Cấu trúc dự án Vue.js (src/, public/, App.vue, main.js).
Thực hành:
Cài Node.js và Vue CLI (nếu chưa có):
bash
Sao chép mã
npm install -g @vue/cli
vue create vue-project
cd vue-project
npm run serve
Khám phá cấu trúc file App.vue và main.js.
Từ khóa để tìm hiểu: Vue.js, Vue CLI, SPA, Single Page Application.
Giờ 2: Data Binding và Directives
Mục tiêu: Hiểu cách Vue.js quản lý dữ liệu và sử dụng directives.
Nội dung học:
Data Binding: {{ }}.
Directives phổ biến: v-bind, v-model, v-if, v-for, v-on.
Thực hành:
Tạo giao diện nhập liệu:
html
Sao chép mã
<template>
  <div>
    <h1>{{ message }}</h1>
    <input v-model="message" />
  </div>
</template>
<script>
export default {
  data() {
    return {
      message: "Hello Vue!",
    };
  },
};
</script>
Thực hành v-for để hiển thị danh sách:
html
Sao chép mã
<ul>
  <li v-for="(item, index) in items" :key="index">{{ item }}</li>
</ul>
Từ khóa để tìm hiểu: v-bind, v-model, v-if, v-for.
Giờ 3: Event Handling và Methods
Mục tiêu: Quản lý sự kiện và xử lý logic trong Vue.js.
Nội dung học:
Event Handling (v-on hoặc @).
Phương thức trong Vue (methods).
Thực hành:
Thêm nút bấm để thay đổi dữ liệu:
html
Sao chép mã
<button @click="increment">Tăng</button>
<p>Giá trị: {{ count }}</p>
javascript
Sao chép mã
data() {
  return { count: 0 };
},
methods: {
  increment() {
    this.count++;
  },
},
Thêm một sự kiện khác như hover, keydown.
Từ khóa để tìm hiểu: Event Handling, Methods, @click.
Giờ 4: Components
Mục tiêu: Tạo và sử dụng các component tái sử dụng.
Nội dung học:
Component là gì? Tại sao cần component?
Cách tạo component.
Truyền dữ liệu qua props và sử dụng emit để gửi sự kiện ngược lại.
Thực hành:
Tạo một component Card hiển thị thông tin sản phẩm:
javascript
Sao chép mã
// Card.vue
<template>
  <div class="card">
    <h3>{{ title }}</h3>
    <p>{{ content }}</p>
  </div>
</template>
<script>
export default {
  props: ["title", "content"],
};
</script>
Sử dụng component trong App.vue:
html
Sao chép mã
<Card title="Sản phẩm 1" content="Mô tả sản phẩm 1" />
Từ khóa để tìm hiểu: Vue Components, Props.
Giờ 5: State Management với Local Storage
Mục tiêu: Lưu trữ dữ liệu cục bộ trong trình duyệt.
Nội dung học:
Lưu và đọc dữ liệu từ Local Storage.
Cập nhật dữ liệu qua Vue.
Thực hành:
Tạo ứng dụng danh sách công việc (To-Do List):
Người dùng có thể thêm/xóa công việc.
Lưu danh sách công việc vào Local Storage.
javascript
Sao chép mã
methods: {
  saveTasks() {
    localStorage.setItem("tasks", JSON.stringify(this.tasks));
  },
  loadTasks() {
    const tasks = localStorage.getItem("tasks");
    if (tasks) this.tasks = JSON.parse(tasks);
  },
},
Từ khóa để tìm hiểu: Local Storage, Vue.js State Management.
Giờ 6: Style và Animation
Mục tiêu: Thêm CSS và hiệu ứng động vào ứng dụng.
Nội dung học:
Thêm style cục bộ trong Vue.
Sử dụng transition để tạo hiệu ứng.
Thực hành:
Tạo hiệu ứng cho danh sách công việc:
html
Sao chép mã
<transition name="fade">
  <li v-for="(item, index) in items" :key="index">{{ item }}</li>
</transition>
css
Sao chép mã
.fade-enter-active, .fade-leave-active {
  transition: opacity 0.5s;
}
.fade-enter, .fade-leave-to {
  opacity: 0;
}
Từ khóa để tìm hiểu: Vue CSS, Vue Transition.
Giờ 7: Vue Router (Tùy chọn không backend)
Mục tiêu: Tạo ứng dụng nhiều trang chỉ với frontend.
Nội dung học:
Cài đặt Vue Router.
Tạo routes đơn giản.
Thực hành:
Tạo hai trang: Trang chính và Trang chi tiết.
javascript
Sao chép mã
const routes = [
  { path: "/", component: Home },
  { path: "/about", component: About },
];
Sử dụng <router-link> để chuyển đổi giữa các trang.
Từ khóa để tìm hiểu: Vue Router, SPA Navigation.
Giờ 8: Dự án nhỏ: To-Do List với Vue
Mục tiêu: Áp dụng toàn bộ kiến thức để xây dựng ứng dụng nhỏ.
Hoạt động:
Xây dựng ứng dụng To-Do List:
Thêm/xóa công việc.
Hiển thị danh sách công việc.
Lưu dữ liệu bằng Local Storage.
Tích hợp hiệu ứng chuyển đổi khi thêm/xóa công việc.
Triển khai ứng dụng lên Netlify hoặc Vercel (tùy chọn).
Lưu ý
Tài liệu tham khảo: Vue.js Documentation.
Thực hành liên tục: Tự tạo thêm các ứng dụng nhỏ như đồng hồ, bộ đếm, hay blog frontend.
Hãy bắt đầu và tận hưởng việc học Vue.js nhé! 🚀
```
```
v-bind:

Ràng buộc thuộc tính hoặc class động vào một giá trị từ dữ liệu hoặc biểu thức.
v-model:

Thiết lập ràng buộc hai chiều giữa dữ liệu và giao diện người dùng (two-way binding).
v-for:

Lặp qua danh sách hoặc đối tượng để tạo các phần tử DOM lặp lại.
v-if, v-else-if, v-else:

Điều kiện hiển thị các phần tử DOM dựa trên giá trị logic.
v-show:

Tương tự v-if, nhưng chỉ thay đổi thuộc tính CSS display thay vì thêm/xóa phần tử DOM.
v-on (hoặc @):

Lắng nghe và xử lý các sự kiện DOM (tương đương với onClick trong React).
v-slot:

Định nghĩa nội dung tùy chỉnh trong thành phần con (slots), tương tự như props.children nhưng linh hoạt hơn.
computed:

Tính toán giá trị dựa trên dữ liệu hiện tại, được tối ưu hóa để giảm thiểu việc tính toán lại (memoized).
watch:

Theo dõi sự thay đổi của dữ liệu và thực thi các phản ứng phụ khi có thay đổi.
ref:

Truy cập trực tiếp vào phần tử DOM hoặc thành phần con trong Vue.
emit:

Gửi sự kiện từ thành phần con lên thành phần cha (thay thế callback trong React).
props:

Truyền dữ liệu từ thành phần cha xuống thành phần con, tương tự React nhưng đơn giản hóa cú pháp.
Directives (Chỉ thị):

Là các từ khóa bắt đầu với v- như v-bind, v-model,... dùng để thêm logic vào template.
setup:

Được sử dụng trong Composition API, tập trung logic và trạng thái của thành phần trong một hàm duy nhất.
template:

Vue sử dụng template để khai báo giao diện người dùng thay vì JSX như trong React.
Filters:

Chuyển đổi dữ liệu trong các template (bị loại bỏ trong Vue 3 nhưng vẫn phổ biến trong Vue 2).
Reactive System:

Vue cung cấp cơ chế phản ứng lại thay đổi dữ liệu (reactivity) tự động mà không cần sử dụng setState như React.
Lifecycle Hooks:

Vue có các hook như mounted, created, updated, và destroyed, tương tự nhưng có tên khác so với React.
Transition & Animation:

Vue hỗ trợ tích hợp sẵn các hiệu ứng chuyển đổi và hoạt hình với các directive như v-enter và v-leave.
Mixins:

Tái sử dụng logic giữa các thành phần (đã được thay thế một phần bởi Composition API trong Vue 3).
```
```
<template>
  <div class="hello">
    <h1>{{ msg }}</h1>
    <h2>Essential Links</h2>
    <table>
      <thead>
        <tr>
          <th>Link Name</th>
          <th>URL</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(link, index) in links" :key="index">
          <td>{{ link.name }}</td>
          <td>
            <a :href="link.url" target="_blank">{{ link.url }}</a>
          </td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<script>
export default {
  name: 'HelloWorld',
  data () {
    return {
      msg: 'Welcome to Your Vue.js App',
      links: [
        { name: 'Core Docs', url: 'https://vuejs.org' },
        { name: 'Forum', url: 'https://forum.vuejs.org' },
        { name: 'Community Chat', url: 'https://chat.vuejs.org' },
        { name: 'Twitter', url: 'https://twitter.com/vuejs' },
        { name: 'Docs for This Template', url: 'http://vuejs-templates.github.io/webpack/' },
        { name: 'vue-router', url: 'http://router.vuejs.org/' },
        { name: 'vuex', url: 'http://vuex.vuejs.org/' },
        { name: 'vue-loader', url: 'http://vue-loader.vuejs.org/' },
        { name: 'awesome-vue', url: 'https://github.com/vuejs/awesome-vue' }
      ]
    }
  }
}
</script>

<style scoped>
h1, h2 {
  font-weight: normal;
}
table {
  width: 100%;
  border-collapse: collapse;
  margin: 20px 0;
}
th, td {
  border: 1px solid #ddd;
  padding: 8px;
  text-align: left;
}
th {
  background-color: #f4f4f4;
}
a {
  color: #42b983;
}
</style>

```
```
<template>
  <div :class="openNavigation ? 'header open' : 'header'">
    <div class="header-container">
      <a class="logo" href="#hero">
        <img :src="brainwave" width="190" height="40" alt="Brainwave" />
      </a>

      <nav :class="openNavigation ? 'navigation open' : 'navigation'">
        <div class="nav-links">
          <a
            v-for="item in navigation"
            :key="item.id"
            :href="item.url"
            @click="handleClick"
            :class="item.url === currentHash ? 'nav-link active' : 'nav-link'"
          >
            {{ item.title }}
          </a>
        </div>

        <HamburgerMenu />
      </nav>

      <a href="#signup" class="signup-link">New account</a>
      <Button class="signin-button" href="#login">Sign in</Button>

      <Button class="menu-toggle" px="px-3" @click="toggleNavigation">
        <MenuSvg :openNavigation="openNavigation" />
      </Button>
    </div>
  </div>
</template>

<script>
import { ref, computed } from 'vue';
import { useRoute } from 'vue-router';
import { disablePageScroll, enablePageScroll } from 'scroll-lock';
import { brainwave } from '../assets';
import { navigation } from '../constants';
import Button from './Button.vue';
import MenuSvg from '../assets/svg/MenuSvg.vue';
import HamburgerMenu from './design/Header.vue';

export default {
  name: 'Header',
  components: {
    Button,
    MenuSvg,
    HamburgerMenu
  },
  setup() {
    const route = useRoute();
    const openNavigation = ref(false);
    const currentHash = computed(() => route.hash);

    const toggleNavigation = () => {
      if (openNavigation.value) {
        openNavigation.value = false;
        enablePageScroll();
      } else {
        openNavigation.value = true;
        disablePageScroll();
      }
    };

    const handleClick = () => {
      if (!openNavigation.value) return;

      enablePageScroll();
      openNavigation.value = false;
    };

    return {
      brainwave,
      navigation,
      openNavigation,
      currentHash,
      toggleNavigation,
      handleClick
    };
  }
};
</script>

<style scoped>
.header {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  z-index: 50;
  background-color: rgba(34, 34, 34, 0.9);
  border-bottom: 1px solid #ccc;
  backdrop-filter: blur(10px);
}

.header.open {
  background-color: #222;
}

.header-container {
  display: flex;
  align-items: center;
  padding: 1rem 2rem;
}

.logo {
  display: block;
  width: 12rem;
  margin-right: 2rem;
}

.navigation {
  display: none;
}

.navigation.open {
  display: flex;
  position: fixed;
  top: 5rem;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: #222;
}

.nav-links {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  margin: auto;
}

.nav-link {
  text-transform: uppercase;
  color: rgba(255, 255, 255, 0.5);
  font-size: 1.25rem;
  padding: 1rem;
  transition: color 0.3s;
}

.nav-link:hover {
  color: #ff6347;
}

.nav-link.active {
  color: #fff;
}

.signup-link {
  display: none;
  color: rgba(255, 255, 255, 0.5);
  transition: color 0.3s;
}

.signup-link:hover {
  color: #fff;
}

.signin-button {
  display: none;
}

.menu-toggle {
  margin-left: auto;
}

@media (min-width: 1024px) {
  .navigation {
    display: flex;
    position: static;
    flex: 1;
    background: transparent;
  }

  .nav-links {
    flex-direction: row;
  }

  .signup-link {
    display: block;
    margin-right: 2rem;
  }

  .signin-button {
    display: flex;
  }

  .menu-toggle {
    display: none;
  }
}
</style>

```
```
1. Cài đặt môi trường
Frontend: Vue.js

Cài Node.js (kèm npm).
Cài Vue CLI: npm install -g @vue/cli.
Tạo dự án Vue mới: vue create login-app.
Backend: .NET (C#)

Cài đặt .NET SDK (tải từ trang chủ Microsoft).
Tạo dự án Web API:
bash
Sao chép mã
dotnet new webapi -n LoginAPI
cd LoginAPI
Cài đặt thư viện hỗ trợ kết nối SQL Server:
bash
Sao chép mã
dotnet add package Microsoft.EntityFrameworkCore.SqlServer
dotnet add package Microsoft.EntityFrameworkCore.Tools
Database: SQL Server

Sử dụng SQL Server Management Studio (SSMS) để quản lý cơ sở dữ liệu.
2. Thiết kế cơ sở dữ liệu
Tạo bảng trong SQL Server cho việc lưu trữ thông tin người dùng:

sql
Sao chép mã
CREATE DATABASE LoginDB;

USE LoginDB;

CREATE TABLE Users (
    Id INT PRIMARY KEY IDENTITY,
    Username NVARCHAR(50) NOT NULL UNIQUE,
    PasswordHash NVARCHAR(255) NOT NULL
);
3. Backend: Xây dựng API với .NET
Cấu hình kết nối database: Thêm chuỗi kết nối vào appsettings.json:

json
Sao chép mã
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=YOUR_SERVER;Database=LoginDB;Trusted_Connection=True;"
  }
}
Tạo Model và DbContext:

csharp
Sao chép mã
using Microsoft.EntityFrameworkCore;

public class User
{
    public int Id { get; set; }
    public string Username { get; set; }
    public string PasswordHash { get; set; }
}

public class AppDbContext : DbContext
{
    public AppDbContext(DbContextOptions<AppDbContext> options) : base(options) { }
    public DbSet<User> Users { get; set; }
}
Thêm API đăng nhập:

Thêm controller: LoginController.
Logic kiểm tra đăng nhập:
csharp
Sao chép mã
[ApiController]
[Route("api/[controller]")]
public class LoginController : ControllerBase
{
    private readonly AppDbContext _context;

    public LoginController(AppDbContext context)
    {
        _context = context;
    }

    [HttpPost("login")]
    public async Task<IActionResult> Login([FromBody] LoginRequest request)
    {
        var user = await _context.Users.FirstOrDefaultAsync(u => u.Username == request.Username);
        if (user == null || !BCrypt.Net.BCrypt.Verify(request.Password, user.PasswordHash))
        {
            return Unauthorized(new { message = "Invalid username or password" });
        }

        return Ok(new { message = "Login successful" });
    }
}

public class LoginRequest
{
    public string Username { get; set; }
    public string Password { get; set; }
}
Chạy dự án:

bash
Sao chép mã
dotnet run
4. Frontend: Xây dựng giao diện với Vue.js
Cài đặt axios để gọi API:

bash
Sao chép mã
npm install axios
Tạo component Login:

File: src/components/Login.vue
vue
Sao chép mã
<template>
  <div class="login-container">
    <form @submit.prevent="login">
      <h2>Login</h2>
      <div>
        <label for="username">Username</label>
        <input id="username" v-model="username" type="text" required />
      </div>
      <div>
        <label for="password">Password</label>
        <input id="password" v-model="password" type="password" required />
      </div>
      <button type="submit">Login</button>
      <p v-if="errorMessage">{{ errorMessage }}</p>
    </form>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  data() {
    return {
      username: '',
      password: '',
      errorMessage: ''
    };
  },
  methods: {
    async login() {
      try {
        const response = await axios.post('http://localhost:5000/api/login/login', {
          username: this.username,
          password: this.password
        });
        alert(response.data.message);
      } catch (error) {
        this.errorMessage = error.response?.data?.message || 'Login failed';
      }
    }
  }
};
</script>

<style>
/* Thêm style tại đây */
</style>
5. Chạy ứng dụng
Frontend:

bash
Sao chép mã
npm run serve
Truy cập http://localhost:8080.

Backend:

.NET API chạy ở http://localhost:5000.
6. Bảo mật nâng cao
Hash mật khẩu: Dùng BCrypt khi lưu mật khẩu vào database.
Token hóa: Sử dụng JWT để quản lý session.
Nếu cần hướng dẫn chi tiết hơn hoặc mở rộng, hãy cho mình biết! 😊
```
```
ể hạn chế mỗi IP máy tính chỉ có thể đăng nhập một lần, bạn có thể sử dụng một cơ chế như sau:

Cách thực hiện
1. Sử dụng Map để quản lý trạng thái đăng nhập theo IP
Dùng một ConcurrentHashMap trên server để lưu trạng thái đăng nhập của từng IP. Key sẽ là địa chỉ IP của máy, và value là thông tin phiên đăng nhập hoặc trạng thái đăng nhập.

java
Sao chép mã
import java.util.concurrent.ConcurrentHashMap;

public class LoginManager {
    private static final ConcurrentHashMap<String, String> loggedInIPs = new ConcurrentHashMap<>();

    public static boolean isLoggedIn(String ipAddress) {
        return loggedInIPs.containsKey(ipAddress);
    }

    public static void login(String ipAddress, String userName) {
        loggedInIPs.put(ipAddress, userName);
    }

    public static void logout(String ipAddress) {
        loggedInIPs.remove(ipAddress);
    }

    public static String getLoggedInUser(String ipAddress) {
        return loggedInIPs.get(ipAddress);
    }
}
2. Kiểm tra trạng thái trước khi đăng nhập
Trong LoginAction, kiểm tra xem IP của người dùng đã tồn tại trong loggedInIPs hay chưa. Nếu tồn tại, không cho phép đăng nhập lại.

Ví dụ trong LoginAction:

java
Sao chép mã
@Override
public ActionForward execute(ActionMapping mapping, ActionForm form,
                             HttpServletRequest request, HttpServletResponse response) throws Exception {
    HttpSession session = request.getSession();
    String userName = request.getParameter("userID");
    String password = request.getParameter("password");
    String ipAddress = request.getRemoteAddr(); // Lấy địa chỉ IP của client

    // Kiểm tra nếu IP đã đăng nhập
    if (LoginManager.isLoggedIn(ipAddress)) {
        request.setAttribute("error", "This IP is already logged in. Please log out first.");
        return mapping.findForward("error");
    }

    // Kiểm tra thông tin đăng nhập (ví dụ: từ cơ sở dữ liệu)
    boolean isValid = checkCredentials(userName, password);
    if (!isValid) {
        request.setAttribute("error", "Invalid username or password.");
        return mapping.findForward("login");
    }

    // Đăng nhập thành công
    LoginManager.login(ipAddress, userName);
    session.setAttribute("user", userName);

    return mapping.findForward("success");
}
3. Xóa trạng thái đăng nhập khi người dùng đăng xuất
Khi người dùng đăng xuất, xóa địa chỉ IP khỏi danh sách trong LoginManager.

Ví dụ trong LogoutAction:

java
Sao chép mã
@Override
public ActionForward execute(ActionMapping mapping, ActionForm form,
                             HttpServletRequest request, HttpServletResponse response) throws Exception {
    HttpSession session = request.getSession(false);
    if (session != null) {
        String ipAddress = request.getRemoteAddr();
        LoginManager.logout(ipAddress);
        session.invalidate();
    }
    return mapping.findForward("login");
}
4. Tự động xử lý khi session hết hạn
Kết hợp với HttpSessionListener để tự động xóa IP khỏi danh sách khi session hết hạn.

Ví dụ:

java
Sao chép mã
@Override
public void sessionDestroyed(HttpSessionEvent event) {
    HttpSession session = event.getSession();
    String ipAddress = (String) session.getAttribute("ipAddress");
    if (ipAddress != null) {
        LoginManager.logout(ipAddress);
    }
}
5. Kiểm tra ở các trang khác (Middleware)
Đảm bảo người dùng không thể truy cập các trang khác mà không đăng nhập bằng cách kiểm tra trạng thái ở mọi Action.

Ví dụ:

java
Sao chép mã
@Override
public ActionForward execute(ActionMapping mapping, ActionForm form,
                             HttpServletRequest request, HttpServletResponse response) throws Exception {
    HttpSession session = request.getSession(false);
    String ipAddress = request.getRemoteAddr();

    // Kiểm tra nếu IP không tồn tại trong danh sách đăng nhập
    if (!LoginManager.isLoggedIn(ipAddress)) {
        response.sendRedirect("login.do");
        return null;
    }

    return mapping.findForward("success");
}
Lợi ích
Quản lý chặt chẽ: Đảm bảo mỗi IP chỉ đăng nhập một lần, phù hợp với yêu cầu.
Dễ mở rộng: Có thể tùy chỉnh để áp dụng thêm các quy tắc hoặc giới hạn theo nhu cầu.
An toàn: Session tự động hủy hoặc bị vô hiệu hóa khi người dùng đóng trình duyệt hoặc không hoạt động.
Lưu ý
Nếu nhiều người dùng có chung IP (ví dụ: qua NAT), cơ chế này có thể hạn chế quá mức.
Nên kiểm tra IP thực tế bằng cách sử dụng X-Forwarded-For nếu có proxy/ngắn dòng.
```
```
1. Phương pháp client-side: Gửi thông báo logout khi đóng tab hoặc trình duyệt
Bạn có thể sử dụng sự kiện beforeunload hoặc unload của trình duyệt để thông báo cho server rằng người dùng đã rời khỏi ứng dụng và thực hiện hành động logout.

JavaScript (Client-side):
javascript
Sao chép mã
window.addEventListener("beforeunload", function () {
    // Gửi yêu cầu logout đến server khi người dùng đóng tab
    navigator.sendBeacon('/logout', JSON.stringify({ tabToken: window.name }));
});
Giải thích:

sendBeacon đảm bảo rằng dữ liệu được gửi đến server ngay cả khi tab bị đóng.
window.name hoặc sessionStorage có thể lưu tabToken để server nhận diện tab đang hoạt động.
2. Phương pháp server-side: Thiết lập thời gian sống của phiên (session timeout)
Trên server, bạn có thể định cấu hình thời gian sống của phiên (session) để tự động hết hạn nếu người dùng không hoạt động trong một khoảng thời gian nhất định.

Java (Server-side):
Trong file web.xml:

xml
Sao chép mã
<session-config>
    <session-timeout>15</session-timeout> <!-- Tự động logout sau 15 phút không hoạt động -->
</session-config>
Nếu người dùng không gửi bất kỳ yêu cầu nào trong khoảng thời gian 15 phút, session sẽ hết hạn và họ sẽ tự động bị logout.
3. Kết hợp cả hai phương pháp:
Bạn có thể kết hợp:

Gửi yêu cầu logout khi đóng tab (client-side).
Cấu hình session timeout trên server để xử lý trường hợp người dùng không hoạt động nhưng không đóng tab.
Hoàn chỉnh:
Client-side (JavaScript):

javascript
Sao chép mã
// Lắng nghe sự kiện đóng tab hoặc trình duyệt
window.addEventListener("beforeunload", function () {
    navigator.sendBeacon('/logout', JSON.stringify({ tabToken: window.name }));
});

// Tạo tabToken riêng cho mỗi tab và lưu vào sessionStorage
if (!window.name) {
    window.name = 'tab_' + new Date().getTime(); // Tạo tabToken duy nhất
}

// Kiểm tra tabToken
console.log("Tab Token:", window.name);
Server-side (Java):

Trong LogoutAction, xử lý yêu cầu logout:

java
Sao chép mã
public class LogoutAction extends Action {
    public ActionForward execute(ActionMapping mapping, ActionForm form,
                                 HttpServletRequest request, HttpServletResponse response) {
        HttpSession session = request.getSession(false);
        if (session != null) {
            session.invalidate(); // Hủy session của người dùng
        }
        return mapping.findForward("login");
    }
}
Cấu hình session timeout trong web.xml:

xml
Sao chép mã
<session-config>
    <session-timeout>15</session-timeout> <!-- Hết hạn sau 15 phút -->
</session-config>
Lưu ý:
TabToken: Đảm bảo mỗi tab có một tabToken duy nhất (có thể sử dụng window.name hoặc UUID).
Session quản lý: Đảm bảo server xóa thông tin tab khỏi HashMap hoặc session khi người dùng logout.
Thời gian sống của session: Không nên đặt quá dài để tránh lạm dụng tài nguyên.
```
```
<!-- login.jsp -->
<html>
<head>
    <title>Login</title>
    <script>
        // Khi trang được load
        window.onload = function() {
            // Kiểm tra xem tabToken đã tồn tại trong sessionStorage chưa
            var token = sessionStorage.getItem("tabToken");
            
            // Nếu chưa có token, tạo token mới
            if (!token) {
                token = new Date().getTime();  // Sử dụng thời gian hiện tại làm token, hoặc UUID.randomUUID()
                sessionStorage.setItem("tabToken", token);  // Lưu token vào sessionStorage
            }
            
            // Đặt giá trị của tabToken vào trường input ẩn trong form
            document.getElementById('tabToken').value = token;
        };
    </script>
</head>
<body>
    <form action="/login" method="post">
        <!-- Các trường input cho username và password -->
        <label for="username">Username:</label>
        <input type="text" name="userID" id="username" />

        <label for="password">Password:</label>
        <input type="password" name="passWord" id="password" />

        <!-- Input ẩn để gửi token -->
        <input type="hidden" name="tabToken" id="tabToken" />

        <button type="submit">Login</button>
    </form>
</body>
</html>

```
```
public class LoginAction extends Action {
    public ActionForward execute(ActionMapping mapping, ActionForm form,
            HttpServletRequest request, HttpServletResponse response) throws IOException {
        
        // Lấy tabToken từ request
        String tabToken = request.getParameter("tabToken");
        System.out.println("tabToken from login form: " + tabToken);

        // Kiểm tra và xử lý token (lưu vào session hoặc xử lý tùy theo yêu cầu)
        HttpSession session = request.getSession();
        if (tabToken != null) {
            session.setAttribute("tabToken", tabToken);  // Lưu tabToken vào session
        }

        // Kiểm tra thông tin đăng nhập
        String userID = request.getParameter("userID");
        String password = request.getParameter("passWord");
        
        boolean isValidUser = loginLogic.handleLogin(userID, password);
        if (isValidUser) {
            session.setAttribute("userName", userID);  // Lưu tên người dùng vào session
            return mapping.findForward("success");
        } else {
            request.setAttribute("errorMessage", "Invalid username or password.");
            return mapping.findForward("failure");
        }
    }
}

```
```
<!-- search.jsp -->
<html>
<head>
    <title>Search</title>
</head>
<body>
    <h2>Welcome ${sessionScope.userName}</h2> <!-- Hiển thị tên người dùng từ session -->
    
    <form action="/search" method="post">
        <label for="customerName">Customer Name:</label>
        <input type="text" name="customerName" id="customerName" />

        <label for="birthday">Birthday:</label>
        <input type="date" name="birthday" id="birthday" />

        <!-- Đặt giá trị của tabToken vào input ẩn -->
        <input type="hidden" name="tabToken" value="${sessionScope.tabToken}" />

        <button type="submit">Search</button>
    </form>
</body>
</html>

```
```
public class SearchAction extends Action {
    public ActionForward execute(ActionMapping mapping, ActionForm form,
            HttpServletRequest request, HttpServletResponse response) throws IOException {
        
        // Lấy tabToken từ request
        String tabToken = request.getParameter("tabToken");
        System.out.println("tabToken from search form: " + tabToken);

        // Kiểm tra và xử lý tabToken, lấy thông tin người dùng từ session
        HttpSession session = request.getSession();
        String userName = (String) session.getAttribute("userName");

        // Kiểm tra nếu có token và người dùng
        if (userName != null) {
            request.setAttribute("userName", userName);
        }

        // Các logic tìm kiếm khác ở đây

        return mapping.findForward("success");
    }
}

```
--------------------------------------------
```
public class SearchAction extends Action {
    private SearchDao searchDao;
    
    public void setSearchDao(SearchDao searchDao) {
        this.searchDao = searchDao;
    }

    public ActionForward execute(ActionMapping mapping, ActionForm form,
            HttpServletRequest request, HttpServletResponse response) throws IOException {
        
        // Lấy tabToken từ request
        String tabToken = request.getParameter("tabToken");
        System.out.println("tabToken from search form: " + tabToken);

        // Kiểm tra nếu có tabToken và xử lý thêm nếu cần
        if (tabToken != null) {
            // Lưu hoặc xử lý tabToken nếu cần
            HttpSession session = request.getSession();
            session.setAttribute("tabToken", tabToken);
        }

        // Các logic tìm kiếm khác ở đây
        // Ví dụ: lấy thông tin khách hàng từ form và tìm kiếm trong database
        String customerName = request.getParameter("customerName");

        // Tiếp tục với các logic khác

        return mapping.findForward("success");
    }
}

```
```
<!-- search.jsp -->
<html>
<head>
    <title>Search</title>
</head>
<body>
    <form action="/search" method="post">
        <!-- Các trường input cho tìm kiếm -->
        <label for="customerName">Customer Name:</label>
        <input type="text" name="customerName" id="customerName" />

        <label for="birthday">Birthday:</label>
        <input type="date" name="birthday" id="birthday" />

        <!-- Đặt giá trị của tabToken vào input ẩn -->
        <input type="hidden" name="tabToken" value="${sessionScope.tabToken}" />

        <button type="submit">Search</button>
    </form>
</body>
</html>

```
```
public class LoginAction extends Action {
    public ActionForward execute(ActionMapping mapping, ActionForm form,
            HttpServletRequest request, HttpServletResponse response) throws IOException {
        
        // Lấy tabToken từ request
        String tabToken = request.getParameter("tabToken");
        System.out.println("tabToken from login form: " + tabToken);

        // Kiểm tra và xử lý token (lưu vào session hoặc xử lý tùy theo yêu cầu)
        HttpSession session = request.getSession();
        if (tabToken != null) {
            session.setAttribute("tabToken", tabToken);  // Lưu tabToken vào session
        }

        // Thực hiện các logic đăng nhập khác, ví dụ: kiểm tra username và password

        return mapping.findForward("success");
    }
}

```
```
<!-- login.jsp -->
<html>
<head>
    <title>Login</title>
    <script>
        // Khi trang được load
        window.onload = function() {
            // Kiểm tra xem tabToken đã tồn tại trong sessionStorage chưa
            var token = sessionStorage.getItem("tabToken");
            
            // Nếu chưa có token, tạo token mới
            if (!token) {
                token = new Date().getTime();  // Sử dụng thời gian hiện tại làm token, hoặc UUID.randomUUID()
                sessionStorage.setItem("tabToken", token);  // Lưu token vào sessionStorage
            }
            
            // Đặt giá trị của tabToken vào trường input ẩn trong form
            document.getElementById('tabToken').value = token;
        };
    </script>
</head>
<body>
    <form action="/login" method="post">
        <!-- Các trường input cho username và password -->
        <label for="username">Username:</label>
        <input type="text" name="userID" id="username" />

        <label for="password">Password:</label>
        <input type="password" name="passWord" id="password" />

        <!-- Input ẩn để gửi token -->
        <input type="hidden" name="tabToken" id="tabToken" />

        <button type="submit">Login</button>
    </form>
</body>
</html>

```
```
package fjs.cs.util;

import javax.servlet.*;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import javax.servlet.http.HttpSession;
import java.io.IOException;

public class SessionFilter implements Filter {

    @Override
    public void init(FilterConfig filterConfig) throws ServletException {}

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
            throws IOException, ServletException {
        HttpServletRequest httpRequest = (HttpServletRequest) request;
        HttpServletResponse httpResponse = (HttpServletResponse) response;

        HttpSession session = httpRequest.getSession();
        String activeUrl = (String) session.getAttribute("activeUrl");
        String requestedUrl = httpRequest.getRequestURI();
        String referer = httpRequest.getHeader("Referer");

        // Bỏ qua các URL không cần xác thực
        if (isPublicResource(requestedUrl)) {
            chain.doFilter(request, response);
            return;
        }

        // Kiểm tra nếu `activeUrl` chưa được thiết lập
        if (activeUrl == null) {
            session.setAttribute("activeUrl", requestedUrl);
            chain.doFilter(request, response);
            return;
        }

        // Nếu Referer không tồn tại, kiểm tra xem URL có phải search.do
        if (referer == null) {
            if (requestedUrl.endsWith("/search.do")) {
                // Nếu là search.do, cho phép tiếp tục
                chain.doFilter(request, response);
                return;
            } else {
                // Ngược lại, chuyển hướng về search.do
                httpResponse.sendRedirect(httpRequest.getContextPath() + "/search.do");
                return;
            }
        }

        // Nếu URL hiện tại khác activeUrl
        if (!requestedUrl.equals(activeUrl)) {
            // Trả về lỗi Not Found nếu người dùng cố truy cập URL khác
            httpResponse.sendError(HttpServletResponse.SC_NOT_FOUND, "Not Found");
            return;
        }

        // Nếu tất cả điều kiện hợp lệ, cho phép tiếp tục
        chain.doFilter(request, response);
    }

    @Override
    public void destroy() {}

    private boolean isPublicResource(String uri) {
        return uri.contains("login.do") ||   // Trang đăng nhập
               uri.contains("register.do") || // Trang đăng ký (nếu có)
               uri.contains("/static/") ||   // CSS, JS, hoặc hình ảnh
               uri.endsWith(".css") ||       // File CSS
               uri.endsWith(".js") ||        // File JS
               uri.endsWith(".jpg") ||       // File JPG
               uri.endsWith(".png");         // File PNG
    }
}

```
```

Để xử lý yêu cầu khi người dùng nhấn Reload (F5) trên trình duyệt và buộc trang được chuyển hướng về search.do, bạn có thể sử dụng cơ chế Filter hoặc kết hợp với cách xử lý trạng thái trong session. Dưới đây là cách triển khai:

1. Sử dụng Filter để kiểm tra trạng thái reload
Bạn có thể thêm logic trong Filter để phát hiện khi người dùng reload trang và tự động chuyển hướng về search.do.

Mã nguồn:
java
Sao chép mã
import javax.servlet.Filter;
import javax.servlet.FilterChain;
import javax.servlet.FilterConfig;
import javax.servlet.ServletException;
import javax.servlet.ServletRequest;
import javax.servlet.ServletResponse;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import javax.servlet.http.HttpSession;
import java.io.IOException;

public class ReloadRedirectFilter implements Filter {

    private static final String ACTIVE_URL_KEY = "activeUrl"; // Key lưu URL hiện tại trong session
    private static final String LAST_ACCESS_TIME_KEY = "lastAccessTime"; // Key lưu thời gian truy cập cuối cùng

    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
        // Khởi tạo filter nếu cần
    }

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
            throws IOException, ServletException {
        HttpServletRequest httpRequest = (HttpServletRequest) request;
        HttpServletResponse httpResponse = (HttpServletResponse) response;

        HttpSession session = httpRequest.getSession();
        String activeUrl = (String) session.getAttribute(ACTIVE_URL_KEY);
        String requestedUrl = httpRequest.getRequestURI();
        Long lastAccessTime = (Long) session.getAttribute(LAST_ACCESS_TIME_KEY);

        if (activeUrl == null || !requestedUrl.equals(activeUrl)) {
            // Nếu không có URL hiện tại trong session, hoặc URL không khớp, lưu URL
            session.setAttribute(ACTIVE_URL_KEY, requestedUrl);
            session.setAttribute(LAST_ACCESS_TIME_KEY, System.currentTimeMillis());
            chain.doFilter(request, response);
        } else {
            // Kiểm tra nếu yêu cầu là reload (dựa trên thời gian truy cập gần nhất)
            long currentTime = System.currentTimeMillis();
            if (lastAccessTime != null && currentTime - lastAccessTime < 500) {
                // Nếu yêu cầu reload (trong khoảng thời gian ngắn), chuyển hướng về search.do
                session.removeAttribute(ACTIVE_URL_KEY);
                httpResponse.sendRedirect(httpRequest.getContextPath() + "/search.do");
            } else {
                // Cập nhật thời gian truy cập và cho phép request
                session.setAttribute(LAST_ACCESS_TIME_KEY, currentTime);
                chain.doFilter(request, response);
            }
        }
    }

    @Override
    public void destroy() {
        // Dọn dẹp filter nếu cần
    }
}
2. Cấu hình Filter trong web.xml
Thêm Filter vào tệp web.xml:

xml
Sao chép mã
<filter>
    <filter-name>ReloadRedirectFilter</filter-name>
    <filter-class>com.yourpackage.ReloadRedirectFilter</filter-class>
</filter>

<filter-mapping>
    <filter-name>ReloadRedirectFilter</filter-name>
    <url-pattern>*.do</url-pattern> <!-- Áp dụng cho tất cả các URL kết thúc bằng .do -->
</filter-mapping>
3. Hoạt động của giải pháp
Tab 1 mở search.do:
ACTIVE_URL_KEY trong session được lưu là search.do.
Người dùng chuyển sang edit.do:
ACTIVE_URL_KEY được cập nhật thành edit.do.
**Người dùng nhấn Reload (F5) trên tab đang mở edit.do:
Filter phát hiện đây là hành động reload và chuyển hướng về search.do.
4. Lưu ý
Dựa vào thời gian:
Sử dụng thời gian (lastAccessTime) để phát hiện hành động reload. Điều này hoạt động tốt với yêu cầu cơ bản, nhưng nếu có các yêu cầu đồng thời, bạn cần xử lý đồng bộ.
Hạn chế ảnh hưởng UX:
Nếu search.do yêu cầu người dùng đăng nhập hoặc thực hiện hành động cụ thể trước, hãy đảm bảo người dùng được chuyển hướng về đúng trạng thái mong muốn.
5. Giải pháp bổ sung: JavaScript
Nếu muốn triển khai một cách đơn giản hơn, bạn có thể kết hợp JavaScript để kiểm soát hành động reload trên trình duyệt:

JavaScript (trong JSP hoặc HTML):
javascript
Sao chép mã
window.addEventListener("beforeunload", function () {
    // Khi người dùng nhấn reload, chuyển hướng về search.do
    window.location.href = "/yourProjectPath/search.do";
});
Tuy nhiên, cách này không đảm bảo hoàn toàn nếu người dùng vô hiệu hóa JavaScript hoặc thực hiện reload bằng các công cụ khác (như developer tools).

Bằng cách sử dụng Filter như trên, bạn có thể đảm bảo logic kiểm soát reload hoàn toàn nằm ở phía server và không phụ thuộc vào client.
```
```
import javax.servlet.Filter;
import javax.servlet.FilterChain;
import javax.servlet.FilterConfig;
import javax.servlet.ServletException;
import javax.servlet.ServletRequest;
import javax.servlet.ServletResponse;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import javax.servlet.http.HttpSession;
import java.io.IOException;

public class SingleUrlFilter implements Filter {

    private static final String ACTIVE_URL_KEY = "activeUrl"; // Key lưu trạng thái URL trong session

    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
        // Khởi tạo filter (nếu cần)
    }

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
            throws IOException, ServletException {
        HttpServletRequest httpRequest = (HttpServletRequest) request;
        HttpServletResponse httpResponse = (HttpServletResponse) response;

        HttpSession session = httpRequest.getSession();
        String activeUrl = (String) session.getAttribute(ACTIVE_URL_KEY); // URL hiện tại đang truy cập
        String requestedUrl = httpRequest.getRequestURI(); // URL người dùng đang yêu cầu

        if (activeUrl == null) {
            // Nếu chưa có URL nào được lưu, lưu URL hiện tại và cho phép truy cập
            session.setAttribute(ACTIVE_URL_KEY, requestedUrl);
            chain.doFilter(request, response);
        } else if (activeUrl.equals(requestedUrl)) {
            // Nếu URL yêu cầu trùng với URL đang mở, cho phép truy cập
            chain.doFilter(request, response);
        } else {
            // Nếu có URL khác đang mở, trả về lỗi 404
            httpResponse.sendError(HttpServletResponse.SC_NOT_FOUND, "Not Found");
        }
    }

    @Override
    public void destroy() {
        // Dọn dẹp filter (nếu cần)
    }
}
Cấu hình Filter trong web.xml
Thêm Filter vào tệp web.xml và áp dụng cho các URL bạn muốn bảo vệ.

xml
Sao chép mã
<filter>
    <filter-name>SingleUrlFilter</filter-name>
    <filter-class>com.yourpackage.SingleUrlFilter</filter-class>
</filter>

<filter-mapping>
    <filter-name>SingleUrlFilter</filter-name>
    <url-pattern>*.do</url-pattern> <!-- Áp dụng cho tất cả các URL kết thúc bằng .do -->
</filter-mapping>
Xóa trạng thái URL khi chuyển đổi
Khi người dùng hoàn thành một thao tác (ví dụ: nhấn vào nút Edit hoặc thực hiện một hành động khác), bạn cần xóa trạng thái URL hiện tại trong session. Điều này đảm bảo họ có thể truy cập URL mới.

Trong action xử lý, thêm đoạn mã sau:

java
Sao chép mã
// Xóa URL hiện tại trong session
HttpSession session = request.getSession();
session.removeAttribute("activeUrl");
Ví dụ: Trong action xử lý edit.do, khi nhấn Edit, bạn xóa activeUrl trước khi chuyển hướng sang trang mới.

Luồng hoạt động
Tab 1: Người dùng truy cập search.do. Lúc này, search.do được lưu trong session là URL hiện tại (activeUrl).
Tab 2: Nếu người dùng copy link search.do vào tab mới, filter sẽ phát hiện rằng activeUrl không trống và không khớp với URL mới. Kết quả trả về 404.
Tab 1 tiếp tục: Người dùng nhấn vào Edit, action edit.do sẽ xóa trạng thái search.do khỏi session và lưu edit.do làm URL mới.

```
```
Để giải quyết yêu cầu của bạn một cách thông minh và thực tế, có thể áp dụng một số kỹ thuật mà các ứng dụng web thường sử dụng trong thực tế để quản lý trạng thái người dùng, đặc biệt là khi liên quan đến các tab trình duyệt và session. Đây là một số giải pháp phổ biến và có tính ứng dụng cao để đáp ứng yêu cầu của bạn:

1. Sử dụng Web Storage (SessionStorage hoặc LocalStorage) kết hợp với Session ID
Cách này tận dụng tính năng của SessionStorage (hoặc LocalStorage) để lưu trữ session ID hoặc một thông tin xác thực khác trong trình duyệt. Mỗi tab sẽ có phiên làm việc riêng biệt, nhưng bạn có thể kiểm tra khi nào các tab khác cố gắng truy cập trang.

Cách hoạt động:
Khi người dùng đăng nhập và truy cập search.do:

Tạo một mã session ID duy nhất khi người dùng đăng nhập (có thể là một token hoặc giá trị UUID).
Lưu session ID này vào SessionStorage (hoặc LocalStorage nếu bạn cần dữ liệu tồn tại lâu hơn).
SessionStorage có tính chất lưu trữ riêng biệt cho mỗi tab, vì vậy không thể chia sẻ dữ liệu giữa các tab.
Kiểm tra session khi vào search.do từ tab khác:

Trước khi cho phép truy cập vào search.do, kiểm tra trong SessionStorage để xác nhận rằng session ID hiện tại có trùng với session ID của tab đang đăng nhập hay không.
Nếu không trùng, trả về lỗi "Not Found".
Khi chuyển sang edit.do từ search.do:

Nếu người dùng chuyển từ search.do sang edit.do, không làm thay đổi session ID trong SessionStorage, chỉ cần tiếp tục duy trì trạng thái đã đăng nhập.
Cách triển khai:
1. Lưu và kiểm tra session ID trên Client (JavaScript)
javascript
Sao chép mã
$(document).ready(function() {
    const sessionId = sessionStorage.getItem('sessionId');
    const currentUrl = window.location.pathname;
    
    // Kiểm tra nếu session ID không tồn tại trong SessionStorage (khi chưa đăng nhập)
    if (!sessionId) {
        alert('Not Found');
        window.location.href = '/login.do'; // Redirect đến trang đăng nhập
    }

    // Kiểm tra nếu URL đã tồn tại trong sessionStorage (trường hợp mở tab mới)
    const currentTabSessionId = sessionStorage.getItem('currentTabSessionId');
    if (currentTabSessionId && currentTabSessionId !== sessionId) {
        alert('Not Found');
        window.location.href = '/404'; // Redirect đến trang lỗi
    }

    // Nếu chưa tồn tại sessionId, lưu nó vào SessionStorage khi người dùng đăng nhập
    if (!sessionId && currentUrl === '/login.do') {
        sessionStorage.setItem('sessionId', 'user-session-id-' + new Date().getTime()); // Tạo sessionId ngẫu nhiên hoặc từ backend
    }

    // Lưu sessionId của tab hiện tại vào SessionStorage để kiểm tra trong các tab khác
    if (!currentTabSessionId) {
        sessionStorage.setItem('currentTabSessionId', sessionId);
    }
});
2. Khi chuyển đến edit.do, xóa session ID khỏi SessionStorage nếu cần:
javascript
Sao chép mã
$(document).ready(function() {
    const sessionId = sessionStorage.getItem('sessionId');
    const currentUrl = window.location.pathname;

    if (currentUrl.includes('edit.do')) {
        // Xóa session ID khi chuyển đến edit.do (nếu cần)
        sessionStorage.removeItem('currentTabSessionId');
    }
});
2. Sử dụng Cookies với SameSite Attribute
Một cách hiệu quả và phổ biến trong thực tế là sử dụng cookies với thuộc tính SameSite để kiểm soát hành vi của cookie khi có nhiều tab. Bạn có thể kiểm soát session cookie và chỉ cho phép truy cập từ những tab cùng domain hoặc session.

Cách hoạt động:
Khi người dùng đăng nhập:

Gửi một cookie có thuộc tính SameSite=Strict hoặc SameSite=Lax khi đăng nhập.
Cookie này sẽ chỉ được gửi trong cùng một tab, ngăn chặn việc gửi cookie đến các tab khác.
Kiểm tra khi truy cập vào search.do:

Khi người dùng cố gắng truy cập search.do từ tab khác, cookie sẽ không được gửi đến server (do SameSite), giúp bảo vệ URL khỏi việc bị truy cập bất hợp pháp.
Khi chuyển sang edit.do:

Nếu người dùng chuyển từ search.do sang edit.do, cookie vẫn được gửi trong cùng một tab và không gặp vấn đề về truy cập trái phép từ các tab khác.
Cách triển khai:
1. Gửi Cookie khi đăng nhập với SameSite
java
Sao chép mã
// Gửi cookie khi đăng nhập trong backend (ví dụ trong Servlet hoặc Filter)
Cookie sessionCookie = new Cookie("sessionId", "user-session-id-" + new Date().getTime());
sessionCookie.setHttpOnly(true);
sessionCookie.setSecure(true); // Chỉ gửi cookie qua HTTPS
sessionCookie.setMaxAge(60 * 60); // 1 giờ
sessionCookie.setPath("/"); // Áp dụng cho toàn bộ ứng dụng

// Đặt SameSite attribute để chỉ cho phép gửi cookie trong cùng một tab
sessionCookie.setAttribute("SameSite", "Strict"); // Hoặc "Lax" nếu cần một ít linh động

response.addCookie(sessionCookie);
2. Kiểm tra Cookie khi vào search.do và edit.do
java
Sao chép mã
// Kiểm tra cookie trong filter hoặc servlet khi truy cập vào search.do
Cookie[] cookies = request.getCookies();
String sessionId = null;

for (Cookie cookie : cookies) {
    if ("sessionId".equals(cookie.getName())) {
        sessionId = cookie.getValue();
        break;
    }
}

// Kiểm tra nếu sessionId không tồn tại hoặc không hợp lệ, trả về lỗi Not Found
if (sessionId == null) {
    response.sendError(HttpServletResponse.SC_NOT_FOUND);
    return;
}
3. Sử dụng WebSocket hoặc Server-Side Session Tracking
Một giải pháp mạnh mẽ và linh hoạt là sử dụng WebSocket hoặc server-side session tracking để theo dõi các kết nối và xác thực người dùng trong thời gian thực.

Cách hoạt động:
Khi người dùng đăng nhập, hệ thống sẽ theo dõi trạng thái đăng nhập của người dùng và giữ trạng thái của người dùng (có thể sử dụng WebSocket để theo dõi và thông báo khi một tab không hợp lệ cố gắng truy cập).
Các tab khác sẽ không thể lấy lại session nếu chưa được xác thực hợp lệ từ server.
4. Dùng UUID hoặc Token-based Authentication (JWT)
Một phương pháp hiện đại và phổ biến là sử dụng UUID hoặc JWT (JSON Web Token) để xác thực phiên làm việc.

Cách hoạt động:
Sau khi đăng nhập, gửi một UUID hoặc JWT từ server xuống client (lưu trữ trong cookie hoặc localStorage/sessionStorage).
Khi người dùng truy cập vào search.do, client sẽ gửi token này để xác thực lại với server.
Nếu token không hợp lệ (từ tab khác), server sẽ trả về lỗi "Not Found".
Tóm tắt các giải pháp:
Sử dụng Web Storage (SessionStorage/LocalStorage):

Kiểm tra session ID và quản lý các tab qua SessionStorage hoặc LocalStorage.
Mỗi tab có một session ID riêng biệt, ngăn ngừa việc sao chép URL qua tab khác.
Sử dụng Cookies với SameSite:

Sử dụng cookies với thuộc tính SameSite để kiểm soát cookie chỉ được gửi trong cùng một tab.
WebSocket hoặc Server-Side Session Tracking:

Theo dõi trạng thái người dùng và session qua WebSocket hoặc các hệ thống theo dõi session phía server.
UUID hoặc JWT Authentication:

Sử dụng UUID hoặc JWT để xác thực người dùng qua các tab, đảm bảo rằng chỉ có tab đã đăng nhập mới có thể truy cập trang.
Các phương pháp này đều đã được áp dụng trong thực tế và giúp bảo vệ ứng dụng web khỏi các cuộc tấn công hoặc truy cập trái phép từ các tab khác.
```
```
Mô tả yêu cầu:
Kiểm tra URL của trang:

Sau khi người dùng đăng nhập và vào trang search.do, kiểm tra nếu người dùng cố gắng mở lại URL đó ở tab mới (hoặc chuyển qua các trang như edit.do), thì không cho phép truy cập lại search.do từ tab khác.
Xóa search.do khỏi danh sách đã truy cập khi vào edit.do:

Khi người dùng truy cập vào edit.do từ search.do, xóa URL search.do khỏi danh sách đã lưu.
Kiểm tra lại khi reload search.do:

Khi reload lại trang search.do, kiểm tra và đảm bảo rằng search.do không có trong session hoặc đã được xóa.
Bước 1: Tạo một Security Filter
Tạo Filter kiểm tra session và URL:
java
Sao chép mã
import javax.servlet.*;
import javax.servlet.http.*;
import java.io.IOException;

public class SecurityFilter implements Filter {
    
    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
        // Initialize filter if needed
    }

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
            throws IOException, ServletException {

        HttpServletRequest httpRequest = (HttpServletRequest) request;
        HttpServletResponse httpResponse = (HttpServletResponse) response;

        HttpSession session = httpRequest.getSession(false);
        String currentUrl = httpRequest.getRequestURI();
        
        // Lấy danh sách URL đã duyệt từ session
        Object visitedUrlsObj = session != null ? session.getAttribute("visitedUrls") : null;
        if (visitedUrlsObj == null) {
            visitedUrlsObj = new HashSet<String>();  // Nếu chưa có, khởi tạo danh sách rỗng
        }
        
        Set<String> visitedUrls = (Set<String>) visitedUrlsObj;

        // Kiểm tra nếu người dùng cố gắng truy cập lại `search.do` từ tab khác
        if (visitedUrls.contains(currentUrl) && !currentUrl.contains("edit.do")) {
            // Nếu URL đã tồn tại, trả về lỗi "Not Found"
            httpResponse.sendError(HttpServletResponse.SC_NOT_FOUND);
            return;
        }

        // Lưu `currentUrl` vào session để theo dõi các URL đã truy cập
        visitedUrls.add(currentUrl);
        session.setAttribute("visitedUrls", visitedUrls);
        
        // Nếu vào trang `edit.do`, xóa `search.do` khỏi danh sách URL đã lưu
        if (currentUrl.contains("edit.do")) {
            visitedUrls.remove("search.do");
            session.setAttribute("visitedUrls", visitedUrls);
        }

        // Tiếp tục chuỗi Filter
        chain.doFilter(request, response);
    }

    @Override
    public void destroy() {
        // Clean-up filter resources if necessary
    }
}
Giải thích mã:
Session quản lý các URL đã truy cập:
visitedUrls: Một tập hợp (Set) lưu trữ các URL mà người dùng đã truy cập trong phiên làm việc.
Mỗi lần người dùng truy cập vào một URL mới (ví dụ: search.do), URL đó sẽ được lưu vào session (visitedUrls).
Kiểm tra truy cập vào các URL:
Trước khi cho phép người dùng truy cập một URL, filter sẽ kiểm tra nếu URL đó đã được lưu trong visitedUrls.
Nếu visitedUrls chứa URL của search.do, và người dùng cố gắng truy cập lại URL từ một tab mới (một tab chưa đăng nhập hoặc chưa duyệt qua), filter sẽ trả về lỗi 404 - Not Found.
Xóa URL khỏi danh sách khi chuyển sang edit.do:
Khi người dùng vào edit.do, nếu search.do có trong danh sách visitedUrls, nó sẽ bị xóa đi để đảm bảo không cho phép người dùng reload lại search.do.
Bước 2: Đăng ký filter trong web.xml
xml
Sao chép mã
<filter>
    <filter-name>SecurityFilter</filter-name>
    <filter-class>com.example.SecurityFilter</filter-class>
</filter>

<filter-mapping>
    <filter-name>SecurityFilter</filter-name>
    <url-pattern>/search.do</url-pattern> <!-- Đăng ký cho URL search.do -->
    <url-pattern>/edit.do</url-pattern>   <!-- Đăng ký cho URL edit.do -->
    <!-- Thêm các URL cần bảo vệ khác nếu cần -->
</filter-mapping>
Bước 3: Hoàn thiện về mặt client-side (optional)
Nếu bạn muốn client-side có thể giúp kiểm tra (hoặc thông báo trước khi gửi request đến server), bạn có thể bổ sung thêm một đoạn mã JavaScript để hỗ trợ lọc các tab mới hoặc kiểm tra session.

javascript
Sao chép mã
$(document).ready(function() {
    const currentUrl = window.location.pathname;
    const visitedUrls = JSON.parse(sessionStorage.getItem('visitedUrls')) || [];

    // Kiểm tra nếu URL đã tồn tại trong sessionStorage
    if (visitedUrls.includes(currentUrl)) {
        alert("Not Found");
        window.location.href = "/404"; // Redirect đến trang lỗi
    } else {
        // Nếu chưa tồn tại, thêm vào sessionStorage
        visitedUrls.push(currentUrl);
        sessionStorage.setItem('visitedUrls', JSON.stringify(visitedUrls));
    }

    // Khi chuyển sang edit.do, xóa search.do khỏi sessionStorage
    if (currentUrl.includes('edit.do')) {
        const updatedUrls = visitedUrls.filter(url => url !== '/search.do');
        sessionStorage.setItem('visitedUrls', JSON.stringify(updatedUrls));
    }
});
```
```
Nếu bạn gặp phải tình huống mà một lần gửi request AJAX lại khiến dữ liệu từ cơ sở dữ liệu (DB) bị load lại hai lần, có thể có một số nguyên nhân phổ biến dẫn đến hành vi này. Cụ thể, một lần gửi AJAX có thể kích hoạt một lần query đến DB từ phía server, nhưng sau đó lại xảy ra một lần query nữa sau khi gửi dữ liệu về từ AJAX.

Dưới đây là một số nguyên nhân và cách khắc phục tình huống này.

Nguyên nhân và giải pháp
1. Trả về dữ liệu từ DB trong cả request và AJAX Response
Nguyên nhân:

Trong ứng dụng của bạn, có thể một lần query DB được thực hiện khi load trang ban đầu, và sau đó khi gửi request AJAX, server lại thực hiện thêm một query nữa để trả về dữ liệu. Điều này dẫn đến việc query dữ liệu từ DB hai lần.
Giải pháp:

Đảm bảo rằng bạn chỉ thực hiện query dữ liệu một lần, và lần gửi AJAX chỉ xử lý việc gửi dữ liệu đã có sẵn, không kích hoạt lại việc query DB.
Ví dụ:
Query DB chỉ một lần trong quá trình load trang.
Khi load trang, server chỉ cần lấy dữ liệu từ DB một lần và trả về dưới dạng response cho lần tải trang đầu tiên.
AJAX chỉ gửi dữ liệu đã có (hoặc gửi dữ liệu mới).
Nếu bạn chỉ cần gửi hoặc lấy một số dữ liệu mới từ phía client, AJAX không cần yêu cầu lại DB.
2. Gọi lại dữ liệu khi AJAX trả về (vấn đề bất đồng bộ)
Nguyên nhân:

Bạn có thể đang thực hiện một query DB trong AJAX để trả dữ liệu cho client, sau đó lại tiếp tục gọi một query DB khác trong phần callback của AJAX mà không cần thiết. Điều này có thể khiến dữ liệu bị load lại hai lần, một lần từ AJAX và một lần từ DB.
Giải pháp:

Kiểm tra mã phía server của bạn và đảm bảo rằng việc query dữ liệu từ DB chỉ xảy ra một lần, khi cần thiết. Không gọi query DB lại trong callback của AJAX nếu không cần thiết.
Ví dụ:
javascript
Sao chép mã
// Gửi request AJAX
$.ajax({
    url: '/api/data',
    method: 'GET',
    success: function(response) {
        console.log("Dữ liệu từ AJAX: ", response);
        // Chỉ xử lý dữ liệu trả về từ AJAX ở đây
    },
    error: function(err) {
        console.log("Lỗi khi gửi AJAX: ", err);
    }
});
Trong ví dụ này, bạn chỉ thực hiện gửi request AJAX mà không gọi lại dữ liệu từ DB trong callback.

3. Xử lý nhiều query DB trong controller hoặc service
Nguyên nhân:

Nếu bạn thực hiện quá nhiều query DB trong cùng một controller hoặc service mà không có điều kiện kiểm tra, có thể dẫn đến việc dữ liệu được truy vấn nhiều lần.
Giải pháp:

Hãy kiểm tra logic trong controller và dịch vụ (services) của bạn. Đảm bảo rằng bạn chỉ gọi query DB khi thật sự cần thiết, và tránh việc gọi lại dữ liệu khi nó đã có sẵn trong session hoặc các biến toàn cục.
Ví dụ:
java
Sao chép mã
// Giả sử đây là controller của bạn trong Java
public String loadData() {
    // Kiểm tra xem dữ liệu đã có sẵn trong session chưa
    if (session.getAttribute("customerData") == null) {
        // Chỉ query DB nếu dữ liệu chưa có trong session
        List<Customer> customers = customerService.getAllCustomers();
        session.setAttribute("customerData", customers);
    }
    return "viewData"; // Trả dữ liệu có sẵn trong session
}
Trong ví dụ trên, dữ liệu chỉ được truy vấn từ DB nếu nó không có sẵn trong session.

4. Tự động load lại dữ liệu sau khi gửi AJAX (Refresh của trang)
Nguyên nhân:

Nếu bạn có cơ chế refresh hoặc reloading dữ liệu sau khi gửi AJAX (ví dụ: reload lại trang hoặc fetch lại dữ liệu từ DB sau khi AJAX hoàn tất), điều này có thể khiến việc query DB bị gọi lại hai lần.
Giải pháp:

Kiểm tra xem bạn có đang reload trang hoặc refresh dữ liệu sau khi AJAX hoàn tất không. Nếu có, hãy ngừng hành động đó hoặc sử dụng một cơ chế chỉ load lại dữ liệu khi thật sự cần thiết.
Ví dụ:
javascript
Sao chép mã
$.ajax({
    url: '/api/data',
    method: 'GET',
    success: function(response) {
        // Xử lý dữ liệu trả về mà không reload lại trang
        // Không gọi lại dữ liệu từ DB ở đây
        updateUI(response);
    }
});
Nếu bạn không cần refresh lại dữ liệu từ DB, chỉ cần update giao diện với dữ liệu đã có từ AJAX.

5. Gửi lại request AJAX do sự kiện (Event) bị lặp lại
Nguyên nhân:

Một sự kiện trong ứng dụng của bạn có thể bị lặp lại và gửi AJAX request nhiều lần. Ví dụ: nếu bạn có sự kiện click hoặc submit, sự kiện này có thể bị kích hoạt nhiều lần khiến request AJAX bị gửi lại nhiều lần.
Giải pháp:

Sử dụng preventDefault() và stopPropagation() trong các sự kiện để ngăn chặn việc gửi request lặp lại.
Ví dụ:
javascript
Sao chép mã
$(document).on('submit', 'form', function (e) {
    e.preventDefault(); // Ngừng hành động mặc định của form
    e.stopPropagation(); // Ngừng sự kiện lan truyền
    sendRequestToServer();
});
Tóm tắt:
Để tránh việc query dữ liệu từ DB nhiều lần, hãy kiểm tra kỹ các lý do sau:

Đảm bảo chỉ query dữ liệu từ DB một lần và không gọi lại khi có dữ liệu đã được lưu trữ.
Đảm bảo không gửi nhiều request AJAX mà không cần thiết.
Kiểm tra sự kiện gửi request để đảm bảo không có sự kiện lặp lại hoặc tái kích hoạt request AJAX.
Nếu vẫn gặp sự cố, kiểm tra kỹ lại flow của request và response trong ứng dụng của bạn, và sử dụng công cụ debug (ví dụ: console logs hoặc network tab trong Developer Tools) để theo dõi những request không mong muốn.
```
```
let isRequestInProgress = false; // Biến kiểm tra việc gửi request

$(document).ready(function () {
    if (!isRequestInProgress) {
        sendRequestToServer();
    }
});

function sendRequestToServer() {
    isRequestInProgress = true; // Đánh dấu việc request đang được gửi

    fetch('/your-endpoint', {
        method: 'GET',
        credentials: 'same-origin' // Đảm bảo cookie được gửi cùng request
    })
    .then(response => response.json())
    .then(data => {
        console.log(data);
    })
    .catch(error => {
        console.error('Error:', error);
    })
    .finally(() => {
        isRequestInProgress = false; // Reset lại flag sau khi request hoàn thành
    });
}

```
```
Cách 5: Sử dụng Token trong URL
Cơ chế:

Sinh token duy nhất cho mỗi tab và thêm token vào URL.
Kiểm tra token trên server trong mỗi request.
Mã JavaScript:
javascript
Sao chép mã
$(document).ready(function () {
    let token = new Date().getTime();
    window.history.pushState({}, document.title, window.location.pathname + "?token=" + token);
});
Mã Server (Java - Servlet):
java
Sao chép mã
String token = request.getParameter("token");
if (token == null || !isValidToken(token)) {
    response.sendError(HttpServletResponse.SC_NOT_FOUND, "Page not found");
}
Giải thích:

Dùng window.history.pushState để thêm token duy nhất vào URL mỗi khi người dùng đăng nhập.
Kiểm tra token trong URL trên server và trả về lỗi 404 Not Found nếu token không hợp lệ.
```
```
Cách 4: Sử dụng Session ID và window.name
Cơ chế:

Sinh một session ID duy nhất cho mỗi tab và lưu vào window.name.
Kiểm tra session ID trên mỗi request để xác định tab hợp lệ.
Mã JavaScript (Lưu vào window.name):
javascript
Sao chép mã
$(document).ready(function () {
    if (!window.name) {
        window.name = 'Session_' + new Date().getTime();
    }
    document.cookie = "sessionId=" + window.name + "; path=/; secure;";
});
Mã Server (Java - Servlet):
java
Sao chép mã
String sessionId = null;
Cookie[] cookies = request.getCookies();
if (cookies != null) {
    for (Cookie cookie : cookies) {
        if ("sessionId".equals(cookie.getName())) {
            sessionId = cookie.getValue();
            break;
        }
    }
}
if (sessionId == null || !isValidSessionId(sessionId)) {
    response.sendError(HttpServletResponse.SC_NOT_FOUND, "Page not found");
}
Giải thích:

window.name được sử dụng để tạo session ID duy nhất cho mỗi tab.
Kiểm tra sessionId trên server để chỉ cho phép người dùng truy cập trong tab đã đăng nhập.
```
```
Cách 3: Dùng LocalStorage và Server Session
Cơ chế:

Dùng localStorage để lưu giá trị duy nhất cho mỗi tab.
Server kiểm tra giá trị lưu trong session đối với mỗi request.
Mã JavaScript:
javascript
Sao chép mã
$(document).ready(function () {
    let sessionId = localStorage.getItem("sessionId");
    if (!sessionId) {
        sessionId = 'Session_' + new Date().getTime();
        localStorage.setItem("sessionId", sessionId);
    }
    document.cookie = "sessionId=" + sessionId + "; path=/; secure;";
});
Mã Server (Java - Servlet):
java
Sao chép mã
String sessionId = null;
Cookie[] cookies = request.getCookies();
if (cookies != null) {
    for (Cookie cookie : cookies) {
        if ("sessionId".equals(cookie.getName())) {
            sessionId = cookie.getValue();
            break;
        }
    }
}
if (sessionId == null || !isValidSession(sessionId)) {
    response.sendError(HttpServletResponse.SC_NOT_FOUND, "Page not found");
}
Giải thích:

Dùng localStorage để lưu giá trị duy nhất cho mỗi tab.
Kiểm tra sessionId từ cookie và đối chiếu với giá trị đã lưu trên server.

```
```
Cách 2: Dùng Token Gắn Liền Với Session
Cơ chế:

Sinh một token duy nhất cho mỗi session và gán token vào mỗi tab.
Kiểm tra token trong mỗi request.
Mã JavaScript (Lưu token vào Session):
javascript
Sao chép mã
$(document).ready(function () {
    let token = sessionStorage.getItem("sessionToken");
    if (!token) {
        token = 'Token_' + new Date().getTime();
        sessionStorage.setItem("sessionToken", token);
    }
    document.cookie = "sessionToken=" + token + "; path=/; secure;";
});
Mã Server (Java - Servlet):
java
Sao chép mã
String token = null;
Cookie[] cookies = request.getCookies();
if (cookies != null) {
    for (Cookie cookie : cookies) {
        if ("sessionToken".equals(cookie.getName())) {
            token = cookie.getValue();
            break;
        }
    }
}
if (token == null || !isValidToken(token)) {
    response.sendError(HttpServletResponse.SC_NOT_FOUND, "Page not found");
}
Giải thích:

Sử dụng sessionStorage để tạo token duy nhất cho từng tab. Sau đó, lưu token này vào cookie.
Mỗi lần gửi request, token này được kiểm tra trên server, và chỉ trả về kết quả nếu token hợp lệ.

```
```
Cách 1: Sử dụng window.name và Cookie
Cơ chế:

Gán một giá trị AppId duy nhất cho mỗi tab khi người dùng đăng nhập.
Lưu AppId vào cookie.
Kiểm tra AppId từ cookie trong mỗi request.
Mã JavaScript:
javascript
Sao chép mã
$(document).ready(function () {
    if (!window.name) {
        window.name = 'SEAppId' + (new Date()).getTime();
    }
    document.cookie = "seAppId=" + window.name + "; path=/; secure;";
});

function checkAppId() {
    let cookies = document.cookie.split('; ');
    let appId = cookies.find(cookie => cookie.startsWith("seAppId="));
    if (appId) {
        return appId.split('=')[1];
    }
    return null;
}
Mã Server (Java - Servlet):
java
Sao chép mã
String currentSeAppId = null;
Cookie[] cookies = request.getCookies();
if (cookies != null) {
    for (Cookie cookie : cookies) {
        if ("seAppId".equals(cookie.getName())) {
            currentSeAppId = cookie.getValue();
            break;
        }
    }
}
if (currentSeAppId == null || !isValidAppId(currentSeAppId)) {
    response.sendError(HttpServletResponse.SC_NOT_FOUND, "Page not found");
}
Giải thích:

Mỗi tab sẽ có một giá trị window.name duy nhất, được sử dụng làm AppId và lưu vào cookie.
Khi người dùng copy liên kết sang tab khác, AppId không trùng khớp sẽ khiến server trả về lỗi 404.
```
```
package fjs.cs.util;

import javax.servlet.*;
import javax.servlet.http.Cookie;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import javax.servlet.http.HttpSession;
import java.io.IOException;

public class SessionFilter implements Filter {

    @Override
    public void init(FilterConfig filterConfig) throws ServletException {}

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
            throws IOException, ServletException {

        HttpServletRequest httpRequest = (HttpServletRequest) request;
        HttpServletResponse httpResponse = (HttpServletResponse) response;

        HttpSession session = httpRequest.getSession();
        // Bỏ qua các URL không cần xác thực
        if (isPublicResource(httpRequest.getRequestURI())) {
            chain.doFilter(request, response);
            return;
        }

        // Kiểm tra nếu session không tồn tại hoặc không có user
        if (session == null || session.getAttribute("userName") == null) {
            httpResponse.sendError(HttpServletResponse.SC_NOT_FOUND, "Resource not found");
            return;
        }
        // Lấy cookie `seAppId` hiện tại
        String currentSeAppId = null;
        Cookie[] cookies = httpRequest.getCookies();
        if (cookies != null) {
            for (Cookie cookie : cookies) {
                if ("seAppId".equals(cookie.getName())) {
                    currentSeAppId = cookie.getValue();
                    break;
                }
            }
        }
        String initialSeAppId = (String) session.getAttribute("initialSeAppId");
//        if (currentSeAppId == null) {
//            // Không tìm thấy cookie `seAppId`, trả về 404
//            httpResponse.sendError(HttpServletResponse.SC_NOT_FOUND, "Cookie seAppId không tồn tại.");
//            return;
//        }

        // Lấy giá trị `seAppId` ban đầu từ session
        System.out.println(currentSeAppId);
        System.out.println(initialSeAppId);
        if (initialSeAppId == null) {
            // Nếu chưa có giá trị ban đầu, lưu giá trị hiện tại vào session
            session.setAttribute("initialSeAppId", currentSeAppId);
            String a = (String) session.getAttribute("initialSeAppId");
            System.out.println("daya la cai dau" +a);
        } else if (!initialSeAppId.equals(currentSeAppId)) {
        	System.out.println("vao day");
            // Nếu giá trị hiện tại khác giá trị ban đầu, trả về 404
            httpResponse.sendError(HttpServletResponse.SC_NOT_FOUND, "Giá trị cookie seAppId đã thay đổi.");
            return;
        }

        // Tiếp tục chuỗi filter
        chain.doFilter(request, response);
    }

    @Override
    public void destroy() {}
    private boolean isPublicResource(String uri) {
        return uri.contains("login.do") ||   // Trang đăng nhập
               uri.contains("register.do") || // Trang đăng ký (nếu có)
               uri.contains("/static/") ||   // CSS, JS, hoặc hình ảnh
               uri.endsWith(".css") ||       // File CSS
               uri.endsWith(".js") ||        // File JS
               uri.endsWith(".jpg") ||       // File JPG
               uri.endsWith(".png");         // File PNG
    }
}

```
```
<script type="text/javascript">
	//Events 
// Sử dụng $(document).ready() để đảm bảo cookie được thiết lập ngay khi trang tải
	$(document).ready(function() {
	    generateWindowID();
	});
	
	function generateWindowID() {
	    // Đảm bảo giá trị window.name được thiết lập ngay khi trang tải
	    if (window.name.indexOf("SEAppId") === -1) {
	        window.name = 'SEAppId' + (new Date()).getTime();
	    }
	    setAppId();
	}
	
	function setAppId() {
	    // Đảm bảo cookie được thiết lập với window.name ngay từ đầu
	    let strCookie = 'seAppId=' + window.name + ';';
	    strCookie += ' path=/';
	
	    if (window.location.protocol.toLowerCase() === 'https:') {
	        strCookie += ' secure;';
	    }
	
	    document.cookie = strCookie;
	    console.log('Giá trị cookie seAppId được lưu: ' + window.name);
	}
```
```
	$(window).ready(function() {generateWindowID()});
	$(window).focus(function() {setAppId()});
	$(window).mouseover(function() {setAppId()});
	
	
	function generateWindowID() {
	    // Thay thế se_appframe() bằng window hoặc hàm đã định nghĩa.
	    if (window.name.indexOf("SEAppId") == -1) {
	        window.name = 'SEAppId' + (new Date()).getTime();
	    }
	    setAppId();
	}

	function setAppId() {
	    // Tạo cookie với giá trị window.name
	    let strCookie = 'seAppId=' + window.name + ';';
	    strCookie += ' path=/';

	    if (window.location.protocol.toLowerCase() == 'https:') {
	        strCookie += ' secure;';
	    }

	    document.cookie = strCookie;
	}
```
```
package fjs.cs.action;

import java.io.IOException;
import java.util.UUID;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import javax.servlet.http.HttpSession;

import org.apache.struts.action.Action;
import org.apache.struts.action.ActionForm;
import org.apache.struts.action.ActionForward;
import org.apache.struts.action.ActionMapping;

import fjs.cs.form.loginForm;
import fjs.cs.logic.loginLogic;

public class loginAction extends Action {
    private loginLogic loginLogic;
	
    public void setLoginLogic(loginLogic loginLogic) {
        this.loginLogic = loginLogic;
    }
	
    public ActionForward execute(ActionMapping mapping, ActionForm form,
                                 HttpServletRequest request, HttpServletResponse response) {

    	loginForm loginForm = (loginForm) form;

        // Lấy thông tin người dùng từ form
        String userID = loginForm.getUserID();
        String passWord = loginForm.getPassWord();
        HttpSession session = request.getSession();
        // Kiểm tra thông tin đăng nhập
        boolean check = loginLogic.handleLogin(userID, passWord);
        
        if (check) {
            // Đăng nhập thành công
        	String userName = loginLogic.saveUserName(userID, passWord);
        	session.setAttribute("userName", userName);
        	// Controller xử lý login thành công
        	String tabToken = (String) session.getAttribute("activeTabToken");
        	if (tabToken == null) {
        	    tabToken = UUID.randomUUID().toString();
        	    session.setAttribute("activeTabToken", tabToken);
        	}
        	try {
				response.sendRedirect("search.do?tabToken=" + tabToken);
			} catch (IOException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}

            return null;
        } else {
            // Đăng nhập thất bại
            request.setAttribute("errorMessage", "Invalid username or password.");
            return mapping.findForward("failure");
        }
    }
}
```
```
import javax.servlet.*;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import javax.servlet.http.HttpSession;
import java.io.IOException;
import java.util.HashSet;
import java.util.Set;

public class SessionFilter implements Filter {

    @Override
    public void init(FilterConfig filterConfig) throws ServletException {}

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
        HttpServletRequest httpRequest = (HttpServletRequest) request;
        HttpServletResponse httpResponse = (HttpServletResponse) response;

        HttpSession session = httpRequest.getSession(false);
        String requestURI = httpRequest.getRequestURI();

        // Bỏ qua các URL không cần xác thực
        if (isPublicResource(requestURI)) {
            chain.doFilter(request, response);
            return;
        }

        // Kiểm tra nếu session không tồn tại hoặc không có user
        if (session == null || session.getAttribute("userName") == null) {
            httpResponse.sendError(HttpServletResponse.SC_NOT_FOUND, "Resource not found");
            return;
        }

        // Lấy token từ URL
        String currentTabToken = httpRequest.getParameter("tabToken");
        if (currentTabToken == null) {
            httpResponse.sendError(HttpServletResponse.SC_NOT_FOUND, "Invalid tab token");
            return;
        }

        // Lấy danh sách token đã sử dụng từ session
        Set<String> usedTokens = (Set<String>) session.getAttribute("usedTokens");
        if (usedTokens == null) {
            usedTokens = new HashSet<>();
            session.setAttribute("usedTokens", usedTokens);
        }

        // Nếu token đã được sử dụng, trả về 404
        if (usedTokens.contains(currentTabToken)) {
            httpResponse.sendError(HttpServletResponse.SC_NOT_FOUND, "Token already in use");
            return;
        }

        // Đánh dấu token này là đang được sử dụng
        usedTokens.add(currentTabToken);

        // Cho phép tiếp tục xử lý request
        chain.doFilter(request, response);
    }

    @Override
    public void destroy() {}

    private boolean isPublicResource(String uri) {
        return uri.contains("login.do") ||   // Trang đăng nhập
               uri.contains("register.do") || // Trang đăng ký (nếu có)
               uri.contains("/static/") ||   // CSS, JS, hoặc hình ảnh
               uri.endsWith(".css") ||       // File CSS
               uri.endsWith(".js") ||        // File JS
               uri.endsWith(".jpg") ||       // File JPG
               uri.endsWith(".png");         // File PNG
    }
}
```
```
        let reloadFlag = "false"; // Giá trị mặc định là false (mở tab mới)

        if (performance.navigation.type == performance.navigation.TYPE_RELOAD) {
            reloadFlag = "true"; // Nếu là reload, gán giá trị "true"
        }

        // Gửi giá trị reloadFlag qua AJAX đến server
        $.ajax({
            url: window.location.pathname,
            type: 'GET',
            data: { reloadFlag: reloadFlag },
            success: function(response) {
                console.log('Server response:', response);
            }
        });
```
```
package fjs.cs.util;

import javax.servlet.*;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import javax.servlet.http.HttpSession;
import java.io.IOException;

public class SessionFilter implements Filter {

    @Override
    public void init(FilterConfig filterConfig) throws ServletException {}

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) 
            throws IOException, ServletException {

        HttpServletRequest httpRequest = (HttpServletRequest) request;
        HttpServletResponse httpResponse = (HttpServletResponse) response;

        HttpSession session = httpRequest.getSession();
        
        // Kiểm tra tham số "reloadFlag" từ request (được gửi từ JavaScript)
        String reloadFlag = httpRequest.getParameter("reloadFlag");
        System.out.println("----------- 1 -------------");
        // Kiểm tra xem người dùng reload hay không
        if ("true".equals(reloadFlag)) {
            System.out.println("This is a reload!");
            // Xử lý reload trang ở đây
        } else if ("false".equals(reloadFlag)) {
            System.out.println("This is a new tab!");
            // Xử lý mở tab mới ở đây
        } else {
            System.out.println("Unable to determine whether this is a reload or a new tab.");
        }
        System.out.println("----------- 2 -------------");
        // Tiếp tục filter chain
        chain.doFilter(request, response);
    }

    @Override
    public void destroy() {}
}

```
```
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Login</title>
</head>
<body>
    <h1>Login</h1>
    <form action="login.do" method="post">
        <label for="username">Username:</label>
        <input type="text" name="username" id="username" required>
        <br>
        <label for="password">Password:</label>
        <input type="password" name="password" id="password" required>
        <br>
        <button type="submit">Login</button>
    </form>

    <c:if test="${not empty error}">
        <p style="color:red">${error}</p>
    </c:if>
</body>
</html>

```
```
package fjs.cs.action;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import javax.servlet.http.HttpSession;
import javax.servlet.http.Cookie;
import org.apache.struts.action.Action;
import org.apache.struts.action.ActionForm;
import org.apache.struts.action.ActionForward;
import org.apache.struts.action.ActionMapping;
import java.util.UUID;

public class LoginAction extends Action {

    @Override
    public ActionForward execute(ActionMapping mapping, ActionForm form,
                                 HttpServletRequest request, HttpServletResponse response) throws Exception {
        HttpSession session = request.getSession();
        String username = request.getParameter("username");
        String password = request.getParameter("password");

        // Kiểm tra đăng nhập
        if ("admin".equals(username) && "password".equals(password)) {
            session.setAttribute("userName", username); // Lưu user vào session

            // Tạo token và lưu vào session
            String tabToken = UUID.randomUUID().toString();
            session.setAttribute("activeTabToken", tabToken);

            // Gửi token qua cookie
            Cookie tokenCookie = new Cookie("tabToken", tabToken);
            tokenCookie.setHttpOnly(true); // Không cho phép truy cập từ JavaScript
            tokenCookie.setPath("/");      // Áp dụng toàn bộ ứng dụng
            response.addCookie(tokenCookie);

            // Redirect đến trang search
            return mapping.findForward("search");
        }

        // Đăng nhập thất bại
        request.setAttribute("error", "Tên đăng nhập hoặc mật khẩu không đúng!");
        return mapping.findForward("login");
    }
}

```
```
package fjs.cs.util;

import javax.servlet.*;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import javax.servlet.http.HttpSession;
import javax.servlet.http.Cookie;
import java.io.IOException;

public class SessionFilter implements Filter {

    @Override
    public void init(FilterConfig filterConfig) throws ServletException {}

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
        HttpServletRequest httpRequest = (HttpServletRequest) request;
        HttpServletResponse httpResponse = (HttpServletResponse) response;

        HttpSession session = httpRequest.getSession(false);

        // Bỏ qua các URL không cần xác thực
        if (isPublicResource(httpRequest.getRequestURI())) {
            chain.doFilter(request, response);
            return;
        }

        // Kiểm tra session
        if (session == null || session.getAttribute("userName") == null) {
            httpResponse.sendError(HttpServletResponse.SC_NOT_FOUND, "Not logged in");
            return;
        }

        // Lấy token từ cookie
        String tabToken = getTokenFromCookies(httpRequest);
        String activeTabToken = (String) session.getAttribute("activeTabToken");

        if (tabToken == null || !tabToken.equals(activeTabToken)) {
            httpResponse.sendError(HttpServletResponse.SC_NOT_FOUND, "Invalid or missing tab token");
            return;
        }

        // Đánh dấu token là đã sử dụng
        session.setAttribute("tokenUsed", true);

        // Cho phép tiếp tục xử lý request
        chain.doFilter(request, response);
    }

    @Override
    public void destroy() {}

    // Lấy token từ cookie
    private String getTokenFromCookies(HttpServletRequest request) {
        if (request.getCookies() != null) {
            for (Cookie cookie : request.getCookies()) {
                if ("tabToken".equals(cookie.getName())) {
                    return cookie.getValue();
                }
            }
        }
        return null;
    }

    // Kiểm tra tài nguyên công khai
    private boolean isPublicResource(String uri) {
        return uri.contains("login.do") ||
               uri.contains("/static/") ||
               uri.endsWith(".css") ||
               uri.endsWith(".js") ||
               uri.endsWith(".jpg") ||
               uri.endsWith(".png");
    }
}

```
```
window.addEventListener("beforeunload", () => {
    fetch("/Hibernate_Spring/logoutTab.do?tabToken=" + currentTabToken, { method: "POST" });
});

```
```
dang nhap 1 lan 
package fjs.cs.util;

import javax.servlet.*;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import javax.servlet.http.HttpSession;
import java.io.IOException;
import java.util.HashSet;
import java.util.Set;

public class SessionFilter implements Filter {

    @Override
    public void init(FilterConfig filterConfig) throws ServletException {}

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
        HttpServletRequest httpRequest = (HttpServletRequest) request;
        HttpServletResponse httpResponse = (HttpServletResponse) response;

        HttpSession session = httpRequest.getSession(false);
        String requestURI = httpRequest.getRequestURI();

        // Bỏ qua các URL không cần xác thực
        if (isPublicResource(requestURI)) {
            chain.doFilter(request, response);
            return;
        }

        // Kiểm tra nếu session không tồn tại hoặc không có user
        if (session == null || session.getAttribute("userName") == null) {
            httpResponse.sendError(HttpServletResponse.SC_NOT_FOUND, "Resource not found");
            return;
        }

        // Lấy token từ URL
        String currentTabToken = httpRequest.getParameter("tabToken");
        if (currentTabToken == null) {
            httpResponse.sendError(HttpServletResponse.SC_NOT_FOUND, "Invalid tab token");
            return;
        }

        // Lấy danh sách token đã sử dụng từ session
        Set<String> usedTokens = (Set<String>) session.getAttribute("usedTokens");
        if (usedTokens == null) {
            usedTokens = new HashSet<>();
            session.setAttribute("usedTokens", usedTokens);
        }

        // Nếu token đã được sử dụng, trả về 404
        if (usedTokens.contains(currentTabToken)) {
            httpResponse.sendError(HttpServletResponse.SC_NOT_FOUND, "Token already in use");
            return;
        }

        // Đánh dấu token này là đang được sử dụng
        usedTokens.add(currentTabToken);

        // Cho phép tiếp tục xử lý request
        chain.doFilter(request, response);
    }

    @Override
    public void destroy() {}

    private boolean isPublicResource(String uri) {
        return uri.contains("login.do") ||   // Trang đăng nhập
               uri.contains("register.do") || // Trang đăng ký (nếu có)
               uri.contains("/static/") ||   // CSS, JS, hoặc hình ảnh
               uri.endsWith(".css") ||       // File CSS
               uri.endsWith(".js") ||        // File JS
               uri.endsWith(".jpg") ||       // File JPG
               uri.endsWith(".png");         // File PNG
    }
}

```
```
            // Đăng nhập thành công
        	String userName = loginLogic.saveUserName(userID, passWord);
        	session.setAttribute("userName", userName);
        	// Controller xử lý login thành công
        	String tabToken = (String) session.getAttribute("activeTabToken");
        	if (tabToken == null) {
        	    tabToken = UUID.randomUUID().toString();
        	    session.setAttribute("activeTabToken", tabToken);
        	}
        	try {
				response.sendRedirect("search.do?tabToken=" + tabToken);
			} catch (IOException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}

            return null;
```
```
<filter>
    <filter-name>springSecurityFilterChain</filter-name>
    <filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>
</filter>
<filter-mapping>
    <filter-name>springSecurityFilterChain</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
@Autowired
private DataSource dataSource;

@Override
protected void configure(AuthenticationManagerBuilder auth) throws Exception {
    auth.jdbcAuthentication()
        .dataSource(dataSource)
        .usersByUsernameQuery("SELECT username, password, enabled FROM users WHERE username = ?")
        .authoritiesByUsernameQuery("SELECT username, authority FROM authorities WHERE username = ?");
}

```
```
Java Configuration (Thay thế spring-security.xml)
Bạn có thể cấu hình Spring Security bằng Java nếu không muốn dùng XML:
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
            .authorizeRequests()
                .antMatchers("/login.do", "/static/**").permitAll() // Không yêu cầu đăng nhập
                .antMatchers("/search.do").hasRole("USER")         // ROLE_USER mới được truy cập
                .antMatchers("/admin/**").hasRole("ADMIN")         // ROLE_ADMIN mới được truy cập
                .anyRequest().authenticated()
                .and()
            .formLogin()
                .loginPage("/login.do")                           // Trang đăng nhập
                .defaultSuccessUrl("/home.do")                   // Sau đăng nhập
                .failureUrl("/login.do?error=true")              // Lỗi đăng nhập
                .and()
            .logout()
                .logoutSuccessUrl("/login.do")                   // Sau khi logout
                .invalidateHttpSession(true);
    }

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.inMemoryAuthentication()
            .withUser("admin").password("{noop}password").roles("ADMIN")
            .and()
            .withUser("user").password("{noop}password").roles("USER");
    }
}

```
```
File spring-security.xml (hoặc cấu hình bằng Java)
Thêm file spring-security.xml trong WEB-INF:
<?xml version="1.0" encoding="UTF-8"?>
<beans:beans xmlns="http://www.springframework.org/schema/security"
             xmlns:beans="http://www.springframework.org/schema/beans"
             xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xsi:schemaLocation="
                http://www.springframework.org/schema/security
                http://www.springframework.org/schema/security/spring-security.xsd
                http://www.springframework.org/schema/beans
                http://www.springframework.org/schema/beans/spring-beans.xsd">

    <!-- Cấu hình xác thực đơn giản (In-Memory Authentication) -->
    <authentication-manager>
        <authentication-provider>
            <user-service>
                <user name="admin" password="{noop}password" authorities="ROLE_ADMIN"/>
                <user name="user" password="{noop}password" authorities="ROLE_USER"/>
            </user-service>
        </authentication-provider>
    </authentication-manager>

    <!-- Cấu hình URL bảo mật -->
    <http auto-config="true" use-expressions="true">
        <!-- Cho phép truy cập không cần đăng nhập -->
        <intercept-url pattern="/login.do" access="permitAll"/>
        <intercept-url pattern="/static/**" access="permitAll"/>

        <!-- Bảo vệ tài nguyên cần đăng nhập -->
        <intercept-url pattern="/search.do" access="hasRole('ROLE_USER')"/>
        <intercept-url pattern="/admin/**" access="hasRole('ROLE_ADMIN')"/>

        <!-- Form login -->
        <form-login login-page="/login.do" default-target-url="/home.do" authentication-failure-url="/login.do?error=true"/>
        
        <!-- Đăng xuất -->
        <logout logout-success-url="/login.do" invalidate-session="true"/>
    </http>
</beans:beans>

```
```
<dependencies>
    <!-- Spring Security -->
    <dependency>
        <groupId>org.springframework.security</groupId>
        <artifactId>spring-security-core</artifactId>
        <version>5.8.1</version>
    </dependency>
    <dependency>
        <groupId>org.springframework.security</groupId>
        <artifactId>spring-security-web</artifactId>
        <version>5.8.1</version>
    </dependency>
    <dependency>
        <groupId>org.springframework.security</groupId>
        <artifactId>spring-security-config</artifactId>
        <version>5.8.1</version>
    </dependency>
</dependencies>

```
```
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns="http://xmlns.jcp.org/xml/ns/javaee" xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd" id="WebApp_ID" version="4.0">
  	<display-name>CustomerSystem_Struts</display-name>
	<context-param>
	    <param-name>javax.servlet.jsp.jstl.fmt.encoding</param-name>
	    <param-value>UTF-8</param-value>
	</context-param>
	    <!-- Spring Context Configuration -->
    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>/WEB-INF/spring/applicationContext.xml</param-value>
    </context-param>
    <filter>
	    <filter-name>SessionFilter</filter-name>
	    <filter-class>fjs.cs.util.SessionFilter</filter-class>
	</filter>
	<filter-mapping>
	    <filter-name>SessionFilter</filter-name>
	    <url-pattern>*.do</url-pattern>
	</filter-mapping>
	    
    <!-- Spring Context Loader Listener -->
    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>

	<welcome-file-list>
	    <welcome-file>index.jsp</welcome-file> <!-- Đặt trang login.do làm welcome file -->
	</welcome-file-list>

    <servlet>
        <servlet-name>action</servlet-name>
        <servlet-class>org.apache.struts.action.ActionServlet</servlet-class>
        <init-param>
            <param-name>config</param-name>
            <param-value>/WEB-INF/struts-config.xml</param-value>
        </init-param>
        <load-on-startup>2</load-on-startup>
    </servlet>

    <servlet-mapping>
        <servlet-name>action</servlet-name>
        <url-pattern>*.do</url-pattern>
    </servlet-mapping>
</web-app>
```
```
package fjs.cs.util;

import javax.servlet.*;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import javax.servlet.http.HttpSession;
import java.io.IOException;

public class SessionFilter implements Filter {

    @Override
    public void init(FilterConfig filterConfig) throws ServletException {}

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
        HttpServletRequest httpRequest = (HttpServletRequest) request;
        HttpServletResponse httpResponse = (HttpServletResponse) response;

        HttpSession session = httpRequest.getSession(false);
        String requestURI = httpRequest.getRequestURI();

        // Bỏ qua các URL không cần xác thực
        if (isPublicResource(requestURI)) {
            chain.doFilter(request, response);
            return;
        }

        // Kiểm tra nếu session không tồn tại hoặc không có user
        if (session == null || session.getAttribute("userName") == null) {
            // Trả về mã lỗi 404
            httpResponse.sendError(HttpServletResponse.SC_NOT_FOUND, "Resource not found");
            return;
        }

        // Tiếp tục xử lý request nếu hợp lệ
        chain.doFilter(request, response);
    }

    @Override
    public void destroy() {}

    // Kiểm tra xem URL có phải là tài nguyên công khai không
    private boolean isPublicResource(String uri) {
        return uri.contains("login.do") ||   // Trang đăng nhập
               uri.contains("register.do") || // Trang đăng ký (nếu có)
               uri.contains("/static/") ||   // CSS, JS, hoặc hình ảnh
               uri.endsWith(".css") ||       // File CSS
               uri.endsWith(".js") ||        // File JS
               uri.endsWith(".jpg") ||       // File JPG
               uri.endsWith(".png");         // File PNG
    }
}



```
```
<script type="text/javascript">
    $(document).ready(function () {
        var selectedColumns = ${t}; // Danh sách cột đã chọn
        console.log("Selected Columns: ", selectedColumns);

        // Định nghĩa colModel
        const columnMapping = {
            "Customer ID": {
                name: "customerID",
                index: "customerID",
                width: 75,
                key: true,
                formatter: "showlink",
                formatoptions: {
                    baseLinkUrl: "edit.do",
                    idName: "id"
                }
            },
            "Customer Name": { name: "customerName", index: "customerName", width: 150 },
            "Sex": { name: "sex", index: "sex", width: 80, formatter: formatSex },
            "Birthday": { name: "birthday", index: "birthday", width: 100 },
            "Address": { name: "address", index: "address", width: 200 },
            "Email": { name: "email", index: "email", width: 200 }
        };

        var colNames = selectedColumns;
        var colModel = selectedColumns.map(column => columnMapping[column]);

        // Biến để lưu trạng thái sắp xếp nhiều cột
        var sortColumns = []; // Danh sách cột
        var sortOrders = [];  // Danh sách thứ tự sắp xếp

        // Tạo jqGrid với multisort
        $("#customerGrid").jqGrid({
            data: ${test},
            datatype: "local",
            colNames: colNames,
            colModel: colModel,
            pager: "#customerPager",
            rowNum: 10,
            multiSort: true, // Kích hoạt multisort
            viewrecords: true,
            height: "auto",
            autowidth: true,
            multiselect: true,
            jsonReader: {
                repeatitems: false,
                root: "rows",
                page: "page",
                total: "total",
                records: "records"
            },
            gridComplete: function () {
                updatePaginationButtons(); // Cập nhật trạng thái nút phân trang
            },
            onSortCol: function (index, order) {
                // Lưu trạng thái multisort
                const colIndex = sortColumns.indexOf(index);
                if (colIndex !== -1) {
                    // Nếu cột đã tồn tại, cập nhật thứ tự sắp xếp
                    sortOrders[colIndex] = order;
                } else {
                    // Nếu cột chưa tồn tại, thêm cột mới
                    sortColumns.push(index);
                    sortOrders.push(order);
                }
            }
        });

        // Cập nhật trạng thái các nút phân trang
        function updatePaginationButtons() {
            const currentPage = $("#customerGrid").jqGrid("getGridParam", "page");
            const lastPage = $("#customerGrid").jqGrid("getGridParam", "lastpage");

            $("#previousPageButton").prop("disabled", currentPage <= 1);
            $("#firstPageButton").prop("disabled", currentPage <= 1);
            $("#nextPageButton").prop("disabled", currentPage >= lastPage);
            $("#lastPageButton").prop("disabled", currentPage >= lastPage);
        }

        // Chuyển trang giữ trạng thái multisort
        $("#firstPageButton").click(function () {
            $("#customerGrid").jqGrid("setGridParam", {
                page: 1,
                sortname: sortColumns.join(","), // Nối danh sách cột
                sortorder: sortOrders.join(",") // Nối danh sách thứ tự sắp xếp
            }).trigger("reloadGrid");
        });

        $("#previousPageButton").click(function () {
            const currentPage = $("#customerGrid").jqGrid("getGridParam", "page");
            if (currentPage > 1) {
                $("#customerGrid").jqGrid("setGridParam", {
                    page: currentPage - 1,
                    sortname: sortColumns.join(","),
                    sortorder: sortOrders.join(",")
                }).trigger("reloadGrid");
            }
        });

        $("#nextPageButton").click(function () {
            const currentPage = $("#customerGrid").jqGrid("getGridParam", "page");
            const lastPage = $("#customerGrid").jqGrid("getGridParam", "lastpage");
            if (currentPage < lastPage) {
                $("#customerGrid").jqGrid("setGridParam", {
                    page: currentPage + 1,
                    sortname: sortColumns.join(","),
                    sortorder: sortOrders.join(",")
                }).trigger("reloadGrid");
            }
        });

        $("#lastPageButton").click(function () {
            const lastPage = $("#customerGrid").jqGrid("getGridParam", "lastpage");
            $("#customerGrid").jqGrid("setGridParam", {
                page: lastPage,
                sortname: sortColumns.join(","),
                sortorder: sortOrders.join(",")
            }).trigger("reloadGrid");
        });

        // Formatter cho cột "Sex"
        function formatSex(cellValue) {
            return cellValue === "0" ? "Male" : "Female";
        }

        // Nút cài đặt header
        $("#settingHeaderButton").click(function () {
            window.location.href = window.location.pathname.substring(0, window.location.pathname.indexOf("/", 2)) + "/upload.do";
        });

        console.log("${selectedColumns}");
    });
</script>

```
```
$(document).ready(function () {
    $("#customerGrid").jqGrid({
        url: 'search.do', // Endpoint server xử lý
        datatype: "json", // Dữ liệu trả về từ server dạng JSON
        mtype: "GET", // GET hoặc POST tùy thuộc vào server
        colNames: ["Customer ID", "Customer Name", "Sex", "Birthday", "Address", "Email"],
        colModel: [
            { name: "customerID", index: "customerID", width: 75, key: true },
            { name: "customerName", index: "customerName", width: 150 },
            { name: "sex", index: "sex", width: 80, formatter: formatSex },
            { name: "birthday", index: "birthday", width: 100 },
            { name: "address", index: "address", width: 200 },
            { name: "email", index: "email", width: 200 }
        ],
        pager: "#customerPager",
        rowNum: 10, // Số dòng trên mỗi trang
        rowList: [10, 20, 30], // Tùy chọn số dòng
        sortname: "customerID",
        sortorder: "asc",
        viewrecords: true,
        height: "auto",
        autowidth: true,
        multiselect: true, // Cho phép chọn nhiều dòng
        jsonReader: {
            root: "rows",
            page: "page",
            total: "total",
            records: "records",
            repeatitems: false,
            id: "customerID"
        },
        caption: "Customer List"
    });

    function formatSex(cellValue) {
        return cellValue === "0" ? "Male" : "Female";
    }

    $("#searchButton").click(function () {
        const searchParams = {
            customerName: $("#customerName").val(),
            sex: $("#sex").val(),
            birthdayFrom: $("#birthdayFrom").val(),
            birthdayTo: $("#birthdayTo").val()
        };

        $("#customerGrid").jqGrid("setGridParam", {
            url: 'search.do',
            datatype: "json",
            postData: searchParams, // Gửi tham số tìm kiếm
            page: 1 // Quay về trang đầu tiên
        }).trigger("reloadGrid");
    });
});

```
```
public class SearchAction extends Action {
    private SearchDao searchDao;

    public void setSearchDao(SearchDao searchDao) {
        this.searchDao = searchDao;
    }

    @Override
    public ActionForward execute(ActionMapping mapping, ActionForm form,
                                 HttpServletRequest request, HttpServletResponse response) throws Exception {
        String customerName = request.getParameter("customerName");
        String sex = request.getParameter("sex");
        String fromBirthday = request.getParameter("birthdayFrom");
        String toBirthday = request.getParameter("birthdayTo");

        int page = Integer.parseInt(request.getParameter("page"));
        int rows = Integer.parseInt(request.getParameter("rows"));

        List<CustomerDto> customerList = searchDao.searchCustomers(customerName, sex, fromBirthday, toBirthday);

        // Pagination logic
        int totalRecords = customerList.size();
        int totalPages = (int) Math.ceil((double) totalRecords / rows);

        int startRow = (page - 1) * rows;
        List<CustomerDto> paginatedList = customerList.subList(
            Math.min(startRow, totalRecords),
            Math.min(startRow + rows, totalRecords)
        );

        // Convert to jqGrid JSON format
        Gson gson = new Gson();
        String json = gson.toJson(new jqGridResult(totalRecords, totalPages, page, paginatedList));

        response.setContentType("application/json");
        response.getWriter().write(json);

        return null; // Vì đây là AJAX, không cần điều hướng
    }
}
	
```
```
public class jqGridResult {
    private int total; // Tổng số trang
    private int page; // Trang hiện tại
    private int records; // Tổng số bản ghi
    private List<?> rows; // Dữ liệu từng dòng

    public jqGridResult(int records, int total, int page, List<?> rows) {
        this.records = records;
        this.total = total;
        this.page = page;
        this.rows = rows;
    }

    // Getters và Setters
}

```
```
$("#customerGrid").jqGrid({
    // ... các cấu hình khác
    onSortCol: function (index, iCol, sortorder) {
        // Lưu trạng thái sort vào sessionStorage (hoặc localStorage)
        const sortState = {
            sortColumn: index,
            sortOrder: sortorder
        };
        sessionStorage.setItem("gridSortState", JSON.stringify(sortState));
    },
});

String sortColumn = request.getParameter("sidx");
String sortOrder = request.getParameter("sord");

// Sử dụng sortColumn và sortOrder trong truy vấn database
List<CustomerDto> sortedList = searchDao.searchCustomersSorted(customerName, sex, fromBirthday, toBirthday, sortColumn, sortOrder);
String lastSortColumn = (String) session.getAttribute("lastSortColumn");
String lastSortOrder = (String) session.getAttribute("lastSortOrder");

// Gửi giá trị này về JSP hoặc sử dụng trong logic backend.
String sortColumn = request.getParameter("sidx"); // Cột sắp xếp
String sortOrder = request.getParameter("sord"); // Hướng sắp xếp

// Lưu vào session
session.setAttribute("lastSortColumn", sortColumn);
session.setAttribute("lastSortOrder", sortOrder);
$(document).ready(function () {
    // Kiểm tra nếu đã lưu trạng thái sort
    const savedSortState = JSON.parse(sessionStorage.getItem("gridSortState"));
    let sortColumn = "customerID"; // Mặc định cột sắp xếp
    let sortOrder = "asc"; // Mặc định hướng sắp xếp

    if (savedSortState) {
        sortColumn = savedSortState.sortColumn;
        sortOrder = savedSortState.sortOrder;
    }

    // Khởi tạo jqGrid
    $("#customerGrid").jqGrid({
        // ... các cấu hình khác
        sortname: sortColumn, // Cột để sắp xếp
        sortorder: sortOrder, // Hướng sắp xếp
    });
});

```
```
$("#customerGrid").jqGrid({
    // Các cấu hình khác
    sortname: "customerID", // Giá trị mặc định
    sortorder: "asc",       // Giá trị mặc định
    onSortCol: function (index, sortorder) {
        sortColumn = index;
        sortOrder = sortorder;
    },
});
$(document).ready(function () {
    let sortColumn = "customerID"; // Cột mặc định để sắp xếp
    let sortOrder = "asc";        // Thứ tự mặc định

    // Khi người dùng thay đổi trạng thái sắp xếp
    $("#customerGrid").on("sortCol", function (e, index, sortorder) {
        sortColumn = index;
        sortOrder = sortorder;
    });

    // Cập nhật trạng thái sắp xếp khi load lại grid
    function reloadGridWithSort() {
        $("#customerGrid").jqGrid("setGridParam", {
            sortname: sortColumn,
            sortorder: sortOrder,
        }).trigger("reloadGrid");
    }

    // Cập nhật các nút phân trang
    function updatePaginationButtons() {
        const currentPage = $("#customerGrid").jqGrid("getGridParam", "page");
        const lastPage = $("#customerGrid").jqGrid("getGridParam", "lastpage");
        $("#previousPageButton").prop("disabled", currentPage <= 1);
        $("#firstPageButton").prop("disabled", currentPage <= 1);
        $("#nextPageButton").prop("disabled", currentPage >= lastPage);
        $("#lastPageButton").prop("disabled", currentPage >= lastPage);
    }

    // Khi phân trang, giữ trạng thái sắp xếp
    $("#customerGrid").jqGrid("setGridParam", {
        onPaging: function () {
            setTimeout(function () {
                reloadGridWithSort();
                updatePaginationButtons();
            }, 100);
        }
    });

    // Thêm các nút điều hướng và gọi reloadGridWithSort
    $("#firstPageButton").click(function () {
        $("#customerGrid").jqGrid("setGridParam", { page: 1 });
        reloadGridWithSort();
    });

    $("#previousPageButton").click(function () {
        const currentPage = $("#customerGrid").jqGrid("getGridParam", "page");
        if (currentPage > 1) {
            $("#customerGrid").jqGrid("setGridParam", { page: currentPage - 1 });
            reloadGridWithSort();
        }
    });

    $("#nextPageButton").click(function () {
        const currentPage = $("#customerGrid").jqGrid("getGridParam", "page");
        const lastPage = $("#customerGrid").jqGrid("getGridParam", "lastpage");
        if (currentPage < lastPage) {
            $("#customerGrid").jqGrid("setGridParam", { page: currentPage + 1 });
            reloadGridWithSort();
        }
    });

    $("#lastPageButton").click(function () {
        const lastPage = $("#customerGrid").jqGrid("getGridParam", "lastpage");
        $("#customerGrid").jqGrid("setGridParam", { page: lastPage });
        reloadGridWithSort();
    });
});

```
```
<script type="text/javascript">
    $(document).ready(function () {
        var selectedColumns = ${t}; // Danh sách cột đã chọn
        console.log("Selected Columns: ", selectedColumns);

        // Định nghĩa colModel
        const columnMapping = {
            "Customer ID": {
                name: "customerID",
                index: "customerID",
                width: 75,
                key: true,
                formatter: "showlink",
                formatoptions: {
                    baseLinkUrl: "edit.do",
                    idName: "id"
                }
            },
            "Customer Name": { name: "customerName", index: "customerName", width: 150 },
            "Sex": { name: "sex", index: "sex", width: 80, formatter: formatSex },
            "Birthday": { name: "birthday", index: "birthday", width: 100 },
            "Address": { name: "address", index: "address", width: 200 },
            "Email": { name: "email", index: "email", width: 200 }
        };

        var colNames = selectedColumns;
        var colModel = selectedColumns.map(column => columnMapping[column]);

        // Biến để lưu trạng thái sắp xếp
        var sortColumn = "customerID"; // Giá trị mặc định
        var sortOrder = "asc";         // Giá trị mặc định

        // Tạo jqGrid
        $("#customerGrid").jqGrid({
            data: ${test},
            datatype: "local",
            colNames: colNames,
            colModel: colModel,
            pager: "#customerPager",
            rowNum: 10,
            sortname: sortColumn,
            sortorder: sortOrder,
            viewrecords: true,
            height: "auto",
            autowidth: true,
            multiselect: true,
            jsonReader: {
                repeatitems: false,
                root: "rows",
                page: "page",
                total: "total",
                records: "records"
            },
            gridComplete: function () {
                updatePaginationButtons(); // Cập nhật trạng thái nút phân trang
            },
            onSortCol: function (index, order) {
                sortColumn = index; // Lưu cột đang được sắp xếp
                sortOrder = order;  // Lưu thứ tự sắp xếp
            }
        });

        // Cập nhật trạng thái các nút phân trang
        function updatePaginationButtons() {
            const currentPage = $("#customerGrid").jqGrid("getGridParam", "page");
            const lastPage = $("#customerGrid").jqGrid("getGridParam", "lastpage");

            $("#previousPageButton").prop("disabled", currentPage <= 1);
            $("#firstPageButton").prop("disabled", currentPage <= 1);
            $("#nextPageButton").prop("disabled", currentPage >= lastPage);
            $("#lastPageButton").prop("disabled", currentPage >= lastPage);
        }

        // Chuyển trang và giữ trạng thái sắp xếp
        $("#firstPageButton").click(function () {
            $("#customerGrid").jqGrid("setGridParam", {
                page: 1,
                sortname: sortColumn,
                sortorder: sortOrder
            }).trigger("reloadGrid");
        });

        $("#previousPageButton").click(function () {
            const currentPage = $("#customerGrid").jqGrid("getGridParam", "page");
            if (currentPage > 1) {
                $("#customerGrid").jqGrid("setGridParam", {
                    page: currentPage - 1,
                    sortname: sortColumn,
                    sortorder: sortOrder
                }).trigger("reloadGrid");
            }
        });

        $("#nextPageButton").click(function () {
            const currentPage = $("#customerGrid").jqGrid("getGridParam", "page");
            const lastPage = $("#customerGrid").jqGrid("getGridParam", "lastpage");
            if (currentPage < lastPage) {
                $("#customerGrid").jqGrid("setGridParam", {
                    page: currentPage + 1,
                    sortname: sortColumn,
                    sortorder: sortOrder
                }).trigger("reloadGrid");
            }
        });

        $("#lastPageButton").click(function () {
            const lastPage = $("#customerGrid").jqGrid("getGridParam", "lastpage");
            $("#customerGrid").jqGrid("setGridParam", {
                page: lastPage,
                sortname: sortColumn,
                sortorder: sortOrder
            }).trigger("reloadGrid");
        });

        // Xử lý formatter cho cột "Sex"
        function formatSex(cellValue) {
            return cellValue === "0" ? "Male" : "Female";
        }

        // Nút cài đặt header
        $("#settingHeaderButton").click(function () {
            window.location.href = window.location.pathname.substring(0, window.location.pathname.indexOf("/", 2)) + "/upload.do";
        });

        console.log("${selectedColumns}");
    });
</script>

```
https://drive.google.com/drive/folders/1bPh22WOHnfjV8E2gzLIvzQr1Bx1uL9nx?usp=sharing
```
editaction
package fjs.cs.action;

import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import javax.servlet.http.HttpSession;

import org.apache.struts.action.Action;
import org.apache.struts.action.ActionErrors;
import org.apache.struts.action.ActionForm;
import org.apache.struts.action.ActionForward;
import org.apache.struts.action.ActionMapping;
import org.apache.struts.action.ActionMessage;

import fjs.cs.dao.SearchDao;
import fjs.cs.dto.CustomerDto;

public class EditAction extends Action {
    private SearchDao searchDao;

    public void setSearchDao(SearchDao searchDao) {
        this.searchDao = searchDao;
    }

    public ActionForward execute(ActionMapping mapping, ActionForm form,
                                 HttpServletRequest request, HttpServletResponse response) {
    	String customerId = request.getParameter("id"); 
    	System.out.println(customerId);
    	HttpSession session = request.getSession();
    	if (customerId != null && customerId != "") {
    		session.setAttribute("screen", "Edit");
    	}
    	else if (customerId == null) {
    		session.setAttribute("screen", "Add new");
    	}
    	String screen = (String) session.getAttribute("screen");
    	System.out.println(screen);
    	if ("Edit".equals(screen)) {
        	CustomerDto customer = searchDao.getCustomerById(Integer.parseInt(customerId));
        	request.setAttribute("customer", customer);
        	String birthday = request.getParameter("birthday");
        	String action = request.getParameter("action");
        	ActionErrors errors = new ActionErrors();
        	if("Save".equals(action)) {
        	    // Kiểm tra định dạng birthday
        	    if (birthday != null && !birthday.matches("^\\d{4}/\\d{2}/\\d{2}$")) {
        	        errors.add("birthday", new ActionMessage("error.birthday.invalid"));
        	    }

        	    // Nếu có lỗi, lưu lỗi và chuyển hướng lại form
        	    if (!errors.isEmpty()) {
        	        saveErrors(request, errors);
        	        return mapping.findForward("edit");
        	    }
        	}
    	} 	
    	return mapping.findForward("edit");
    }
}

```
```
<%@ taglib uri="http://struts.apache.org/tags-html" prefix="html" %>
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c" %>
<%@ taglib uri="http://struts.apache.org/tags-html" prefix="html" %>
<%@ taglib uri="/WEB-INF/struts-bean.tld" prefix="bean" %>
<%@ taglib uri="/WEB-INF/struts-logic.tld" prefix="logic" %>
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>Edit Customer</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            background-color: #f5f5f5;
            margin: 0;
            padding: 20px;
        }

        .container {
            max-width: 600px;
            margin: 0 auto;
            background-color: white;
            padding: 20px;
            border-radius: 8px;
            box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
        }

        h1 {
            text-align: center;
        }

        .form-group {
            display: flex;
            justify-content: space-between;
            align-items: center;
            margin-bottom: 20px;
        }

        .form-group label {
            flex: 1;
            margin-right: 10px;
            font-weight: bold;
        }

        .form-group input[type="text"], 
        .form-group input[type="email"], 
        .form-group select, 
        .form-group textarea {
            flex: 2;
            padding: 8px;
            border: 1px solid #ccc;
            border-radius: 4px;
        }

        textarea {
            resize: none;
        }

        .button-group {
            display: flex;
            justify-content: space-between;
            gap: 10px;
        }

        button {
            flex: 1;
            padding: 10px;
            background-color: #4CAF50;
            color: white;
            border: none;
            border-radius: 4px;
            cursor: pointer;
            font-size: 16px;
        }

        button:hover {
            background-color: #45a049;
        }

        .clear-button {
            background-color: #f44336;
        }

        .clear-button:hover {
            background-color: #e53935;
        }

        .error-message {
            color: red;
            text-align: center;
        }

        /* Breadcrumb and user welcome styling */
        .breadcrumb {
            padding: 10px 0;
            font-size: 14px;
        }

        .breadcrumb a {
            color: blue;
            text-decoration: none;
        }

        .breadcrumb a:hover {
            text-decoration: underline;
        }

        .welcome-user-container {
            display: flex;
            justify-content: space-between;
            align-items: center;
            margin-bottom: 20px;
        }

        .divider {
            height: 2px;
            width: 100%;
            background-color: #ccc;
            margin: 20px 0;
        }
    </style>
</head>
<body>
    <div class="container">
        <!-- Breadcrumb and User Welcome Section -->
        <div class="breadcrumb"><a href="/Hibernate_Spring/search.do">Home</a> > Edit Customer</div>
		<div class="divider">${screen}</div>
        <div class="welcome-user-container">
            <p>Welcome, ${user}</p>
            <form action="search" method="post" style="display: inline;">
                <input type="hidden" name="action" value="logout" />
                <button type="submit">Log Out</button>
            </form>
        </div>



        <html:form action="/edit" method="post">
            <h1>Edit Customer</h1>
            <c:if test="${not empty errorMessage}">
                <p class="error-message">${errorMessage}</p>
            </c:if>

            <div class="form-group">
                <label for="id">Customer ID:</label>
                <input type="text" id="id" name="id" value="${customer.customerID}" readonly />
            </div>

            <div class="form-group">
                <label for="name">Customer Name:</label>
                <input type="text" id="name" name="name" value="${customer.customerName}" />
            </div>

            <div class="form-group">
                <label for="email">Email:</label>
                <input type="email" id="email" name="email" value="${customer.email}" />
            </div>

            <div class="form-group">
                <label for="sex">Sex:</label>
                <select id="sex" name="sex">
                    <option value="M" ${customer.sex == 'M' ? 'selected' : ''}>Male</option>
                    <option value="F" ${customer.sex == 'F' ? 'selected' : ''}>Female</option>
                </select>
            </div>

            <div class="form-group">
                <label for="birthday">Birthday:</label>
                <input type="text" id="birthday" name="birthday" value="${customer.birthday}" />
            </div>

            <div class="form-group">
                <label for="address">Address:</label>
                <textarea id="address" name="address" rows="3">${customer.address}</textarea>
            </div>

            <!-- Group of buttons: Save and Clear -->
            <div class="button-group">
                <button type="submit" name="action" value="Save">Save</button>
                <button type="reset" class="clear-button">Clear</button>
            </div>
            <html:errors/>
        </html:form>
    </div>
</body>
</html>

```
```
<%@ taglib uri="http://struts.apache.org/tags-html" prefix="html" %>
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c" %>
<%@ taglib uri="/WEB-INF/struts-bean.tld" prefix="bean" %>
<%@ taglib uri="/WEB-INF/struts-logic.tld" prefix="logic" %>
<html>
<head>
    <title>Setting Header</title>
    <style>
        .selected {
            background-color: lightblue; /* Màu nền khi được chọn */
        }
        label {
            display: block; /* Hiển thị mỗi label trên 1 dòng */
            padding: 5px;
            cursor: pointer;
        }
        .list-container {
            display: flex;
            justify-content: space-between;
        }
        .list {
            width: 200px;
            height: 200px;
            border: 1px solid black;
            padding: 10px;
            overflow-y: auto;
        }
    </style>
    <script>
	    function checkButtonState() {
	        var customerList = document.getElementById('customerList');
	
	            // Nếu label được chọn là đầu tiên, disable Move Up
	            document.getElementById('moveUpBtn').disabled = (selectedLabel.previousElementSibling === null);
	
	            // Nếu label được chọn là cuối cùng, disable Move Down
	            document.getElementById('moveDownBtn').disabled = (selectedLabel.nextElementSibling === null);
	
	            // Các nút Move Left và Move Right chỉ cần kiểm tra nếu có label được chọn
	            document.getElementById('moveLeftBtn').disabled = false;
	            document.getElementById('moveRightBtn').disabled = false;
	    }
        function toggleMoveButtons() {
            var selectedLabel = document.querySelector('#customerList label.selected');
            var moveUpButton = document.getElementById('moveUpButton');
            var moveDownButton = document.getElementById('moveDownButton');

            if (!selectedLabel) {
                return;
            }

            // Disable 'Move Up' if at the top
            if (!selectedLabel.previousElementSibling) {
                moveUpButton.disabled = true;
            } else {
                moveUpButton.disabled = false;
            }

            // Disable 'Move Down' if at the bottom
            if (!selectedLabel.nextElementSibling) {
                moveDownButton.disabled = true;
            } else {
                moveDownButton.disabled = false;
            }
        }
	    function selectLabel(label) {
	        // Bỏ chọn tất cả các label trong cả hai danh sách
	        var allLabels = document.querySelectorAll('.list label');
	        allLabels.forEach(function(item) {
	            item.classList.remove('selected');
	        });
	        // Chọn label hiện tại
	        label.classList.add('selected');
	    }

        function moveUp() {
            var selectedLabel = document.querySelector('#customerList label.selected');
            if (selectedLabel && selectedLabel.previousElementSibling) {
                selectedLabel.parentNode.insertBefore(selectedLabel, selectedLabel.previousElementSibling);
                toggleMoveButtons();
            }
        }

        function moveDown() {
            var selectedLabel = document.querySelector('#customerList label.selected');
            if (selectedLabel && selectedLabel.nextElementSibling) {
                selectedLabel.parentNode.insertBefore(selectedLabel.nextElementSibling, selectedLabel);
            }
        }

        function moveRight() {
            var selectedLabel = document.querySelector('#columnList label.selected');
            if (selectedLabel) {
                document.getElementById('customerList').appendChild(selectedLabel);
            }
        }

        function moveLeft() {
            var selectedLabel = document.querySelector('#customerList label.selected');
            if (selectedLabel) {
                document.getElementById('columnList').appendChild(selectedLabel);
            } else {
                alert('Please select a label to move left.');
            }
        }
        function saveSelectedColumns() {
            var selectedColumns = [];
            var customerList = document.getElementById('customerList');
            var labels = customerList.querySelectorAll('label');
            
            labels.forEach(function(label) {
                selectedColumns.push(label.textContent.trim());
            });
            
            document.getElementById('selectedColumns').value = selectedColumns.join(',');
        }
    </script>
</head>
<body>
    <h2>Manage Search Screen Columns</h2>

    <html:form action="/upload">
        <div class="list-container">
            <!-- Column List -->
            <div id="columnList" class="list">
                <logic:iterate id="column" name="availableColumns">
                    <label onclick="selectLabel(this)">
                        <bean:write name="column" />
                    </label>
                </logic:iterate>
            </div>

            <!-- Move Buttons -->
            <div>
                <button type="button" onclick="moveUp()" >&#8594;</button><br><br>
                <button type="button" onclick="moveDown()">Move Down</button><br><br>
                <button type="button" onclick="moveRight()">Move Right</button><br><br>
                <button type="button" onclick="moveLeft()">Move Left</button>
            </div>

            <!-- Customer List -->
            <div id="customerList" class="list">
                <c:if test="${not empty sessionScope.selectedColumns}">
                    <c:forEach var="selectedColumn" items="${sessionScope.selectedColumns}">
                        <label onclick="selectLabel(this)">
                            <bean:write name="selectedColumn" />
                        </label>
                    </c:forEach>
                </c:if>
            </div>
        </div>

        <!-- Hidden input to store the selected columns -->
        <input type="hidden" name="selectedColumns" id="selectedColumns" 
               value="<%= request.getSession().getAttribute("selectedColumns") != null ? 
                       String.join(",", (String[]) request.getSession().getAttribute("selectedColumns")) : "conchonay" %>" />

        <br><br>
        <button type="submit" onclick="saveSelectedColumns()" name=action value="save">save</button>
    </html:form>
</body>
</html>

```
```
settingaction
package fjs.cs.action;

import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import javax.servlet.http.HttpSession;

import org.apache.struts.action.Action;
import org.apache.struts.action.ActionForm;
import org.apache.struts.action.ActionForward;
import org.apache.struts.action.ActionMapping;

import fjs.cs.dao.SearchDao;

public class UploadAction extends Action {
    private SearchDao searchDao;

    public void setSearchDao(SearchDao searchDao) {
        this.searchDao = searchDao;
    }

    public ActionForward execute(ActionMapping mapping, ActionForm form,
                                 HttpServletRequest request, HttpServletResponse response) {
        // Danh sách tất cả các cột
    	HttpSession session = request.getSession();
        String[] allColumns = {"Customer ID", "Customer Name", "Sex", "Birthday", "Address", "Email"};
        String selectedColumns = request.getParameter("selectedColumns");
        String action = request.getParameter("action");
        String[] avaiDefault = (String[]) session.getAttribute("availableColumns");
        if(avaiDefault==null) {
            List<String> defaultColumns = Arrays.asList("Customer ID", "Customer Name", "Sex", "Birthday", "Address");

            // Các cột còn lại (bao gồm "Email") sẽ là cột chưa được chọn
            List<String> availableList = new ArrayList<>();
            for (String column : allColumns) {
                if (!defaultColumns.contains(column)) {
                    availableList.add(column);
                }
            }

            // Lưu danh sách cột chưa được chọn vào session
            session.setAttribute("availableColumns", availableList.toArray(new String[0]));
        }
        List<String> selectedList = new ArrayList<>();
        List<String> availableList = new ArrayList<>();
        if("save".equals(action)) {
            // Chuyển danh sách cột đã chọn thành danh sách
            if (selectedColumns != null && !selectedColumns.isEmpty()) {
                selectedList = Arrays.asList(selectedColumns.split(","));
                // Lưu danh sách cột đã chọn vào session
                request.getSession().setAttribute("selectedColumns", selectedList.toArray(new String[0]));               
            }

            // Các cột chưa được chọn (cột ẩn) = Tất cả các cột - Cột đã chọn
            for (String column : allColumns) {
                if (!selectedList.contains(column)) {
                    availableList.add(column);
                    System.out.println(column);
                }
            }

            // Lưu danh sách cột ẩn vào session
            request.getSession().setAttribute("availableColumns", availableList.toArray(new String[0]));
            // Chuyển đến trang JSP setting header
            return mapping.findForward("search");
        }
//        else {
//            // Nếu action không phải là "save", thiết lập availableColumns là mảng rỗng
//            session.setAttribute("availableColumns", new String[0]);
//        }
        return mapping.findForward("success");
    }
}

```
```
searchjsp
<%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8"%>
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c"%>
<%@ taglib uri="http://struts.apache.org/tags-html" prefix="html" %>
<%@ taglib uri="/WEB-INF/struts-bean.tld" prefix="bean" %>
<%@ taglib uri="/WEB-INF/struts-logic.tld" prefix="logic" %>
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>Search Customers</title>
    <style type="text/css">
        body {
            margin-left: 20px;
            margin-right: 20px;
            background-color: #bcffff;
        }
        .header {
            border-bottom: 2px solid;
        }
        .header h1 {
            color: red;
        }
        .divider {
            height: 20px;
            width: 100%;
            background-color: #3097ff;
        }
        .search {
            display: flex;
            justify-content: space-around;
            align-items: center;
            padding: 10px;
            margin-top: 20px;
            background-color: #ffff56;
        }
		table {
		    width: 100%;
		    border-collapse: collapse;
		    table-layout :fixed;
		    border :  1px solid green;
		}
		#gbox_customerGrid {
			border :  2px solid green;
			border-collapse: collapse;
		}
		th,
		td {
		    padding: 8px;
		    text-align: left;
		    border-bottom: 1px solid #ddd;
		}
		th {
		    background-color: #f2f2f2;
		}
		tr:first-child > th {
		    background-color: rgb(0, 223, 0);
		    
		}
		tr:nth-child(even) {
		    background-color: #f2f2f2;
		}
    </style>
    
    <!-- Libraries for jqGrid and jQuery -->
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/jqueryui/1.12.1/jquery-ui.min.css">
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/free-jqgrid/4.15.5/css/ui.jqgrid.min.css">
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jquery/3.6.0/jquery.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jqueryui/1.12.1/jquery-ui.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/free-jqgrid/4.15.5/jquery.jqgrid.min.js"></script>
</head>
<body>
    <div class="header">
        <h1>TRAINING</h1>
        <html:errors/>
    </div>
    <div class="container">
    <html:form action="/search" method="post">
		<div class="search">
    <input type="text" id="customerName" name="customerName" placeholder="Customer Name"> <!-- Thêm name -->
    <select id="sex" name="sex"> <!-- Thêm name -->
        <option value=""></option>
        <option value="0">Male</option>
        <option value="1">Female</option>
    </select>
    <input type="text" id="birthdayFrom" name="birthdayFrom" placeholder="Birthday From (YYYY-MM-DD)"> <!-- Thêm name -->
    <input type="text" id="birthdayTo" name="birthdayTo" placeholder="Birthday To (YYYY-MM-DD)"> <!-- Thêm name -->
		    <button id="searchButton" name="action" value="search" type="submit" >Search</button>
		    <button id="deleteButton" name="action" value="delete" type="submit">Delete Selected</button>
		    <button id="firstPageButton">First &lt;&lt;</button>
		    <button id="previousPageButton">Previous &lt;</button>
		    <button id="nextPageButton">Next &gt;</button>
			<button id="edit" name="action" value="edit" type="submit">edit</button>
		    <button id="">Last &gt;&gt;</button>
		    <button id="settingHeaderButton" name="action" value="upload" type="submit">Setting Header</button>
		    <input type="hidden" id="customerIds" name="customerIds" value="">
		</div>
	</html:form>

<table id="customerGrid"></table>
<div id="customerPager" style="display:none"></div>
<script type="text/javascript">
    $(document).ready(function () {
        var selectedColumns = ${t}; // Ví dụ: ["Customer ID", "Customer Name", "Sex", "Birthday", "Address", "Email"]
        console.log("Selected Columns: ", selectedColumns);

        // Tạo colNames và colModel động dựa trên selectedColumns
	    // Định nghĩa mapping object cho colModel
	    const columnMapping = {
        		
	    	    "Customer ID": {
	    	        name: "customerID",
	    	        index: "customerID",
	    	        width: 75,
	    	        key: true,
	    	        formatter: "showlink", // Sử dụng formatter "showlink"
	    	        formatoptions: {
	    	            baseLinkUrl: "edit.do", // Đường dẫn cơ bản
	    	            idName: "id" // Tên tham số query string
	    	        }
	    	    },
	        "Customer Name": { name: "customerName", index: "customerName", width: 150 },
	        "Sex": { name: "sex", index: "sex", width: 80, formatter: formatSex },
	        "Birthday": { name: "birthday", index: "birthday", width: 100 },
	        "Address": { name: "address", index: "address", width: 200 },
	        "Email": { name: "email", index: "email", width: 200 }
	    };
	
	    // Tạo colNames và colModel động dựa trên selectedColumns
	    var colNames = selectedColumns;
	    var colModel = selectedColumns.map(column => columnMapping[column]);

        // Cấu hình jqGrid
        $("#customerGrid").jqGrid({
            data: ${test}, // Dữ liệu
            datatype: "local",
            mtype: "POST",
            colNames: colNames, // Header cột động
            colModel: colModel, // Mô hình cột động
            pager: "#customerPager",
            rowNum: 10,
            sortname: "customerID",
            sortorder: "asc",
            viewrecords: true,
            height: "auto",
            autowidth: true,
            multiselect: true, // Cho phép chọn nhiều hàng
            jsonReader: {
                repeatitems: false,
                root: "rows",
                page: "page",
                total: "total",
                records: "records"
            },
            gridComplete: function () {
                updatePaginationButtons(); // Gọi hàm để cập nhật nút phân trang
            }
        });
        // Kiểm tra trạng thái các nút phân trang
        function updatePaginationButtons() {
            const currentPage = $("#customerGrid").jqGrid("getGridParam", "page");
            const lastPage = $("#customerGrid").jqGrid("getGridParam", "lastpage");

            // Disable nút Previous nếu ở trang đầu tiên
            $("#previousPageButton").prop("disabled", currentPage <= 1);
            $("#firstPageButton").prop("disabled", currentPage <= 1);
            // Disable nút Next nếu ở trang cuối cùng
            $("#nextPageButton").prop("disabled", currentPage >= lastPage);
            $("#lastPageButton").prop("disabled", currentPage >= lastPage);
        }

        // Gọi hàm updatePaginationButtons sau mỗi lần phân trang
        $("#customerGrid").jqGrid("setGridParam", {
            onPaging: function () {
                setTimeout(updatePaginationButtons, 100); // Delay ngắn để cập nhật sau khi phân trang
            }
        });


        // Delete button functionality
	    $("#deleteButton").click(function (event) {
	        const selectedIds = $("#customerGrid").jqGrid('getGridParam', 'selarrrow');
	        if (selectedIds.length === 0) {
	            event.preventDefault(); // Ngăn gửi form nếu không có gì được chọn
	            alert("Please select at least one customer to delete.");
	            return;
	        }
	        // Gán danh sách ID đã chọn vào input ẩn
	        $("#customerIds").val(selectedIds.join(","));
	    });

        // Pagination controls
        $("#firstPageButton").click(function () {
            $("#customerGrid").jqGrid("setGridParam", { page: 1 }).trigger("reloadGrid");
        });

        $("#previousPageButton").click(function () {
            const currentPage = $("#customerGrid").jqGrid("getGridParam", "page");
            if (currentPage > 1) {
                $("#customerGrid").jqGrid("setGridParam", { page: currentPage - 1 }).trigger("reloadGrid");
            }
        });

        $("#nextPageButton").click(function () {
            const currentPage = $("#customerGrid").jqGrid("getGridParam", "page");
            const lastPage = $("#customerGrid").jqGrid("getGridParam", "lastpage");
            if (currentPage < lastPage) {
                $("#customerGrid").jqGrid("setGridParam", { page: currentPage + 1 }).trigger("reloadGrid");
            }
        });

        $("#lastPageButton").click(function () {
            const lastPage = $("#customerGrid").jqGrid("getGridParam", "lastpage");
            $("#customerGrid").jqGrid("setGridParam", { page: lastPage }).trigger("reloadGrid");
        });

        // Formatter for Sex column
        function formatSex(cellValue) {
            return cellValue === "0" ? "Male" : "Female";
        }
        $("#settingHeaderButton").click(function () {
            window.location.href = window.location.pathname.substring(0, window.location.pathname.indexOf("/", 2)) + "/upload.do";
        });
        console.log("${selectedColumns}")
    });
</script>


    </div>
</body>
</html>

```
```
searchaction
package fjs.cs.action;

import java.text.ParseException;
import java.text.SimpleDateFormat;
import java.util.Date;
import java.util.List;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import javax.servlet.http.HttpSession;

import org.apache.struts.action.Action;
import org.apache.struts.action.ActionErrors;
import org.apache.struts.action.ActionForm;
import org.apache.struts.action.ActionForward;
import org.apache.struts.action.ActionMapping;
import org.apache.struts.action.ActionMessage;
import org.apache.struts.action.ActionMessages;

import com.google.gson.Gson;

import fjs.cs.dao.SearchDao;
import fjs.cs.dto.CustomerDto;
import fjs.cs.form.SearchForm;

public class SearchAction extends Action {
    
    private SearchDao searchDao;
	
    public void setSearchDao(SearchDao searchDao) {
        this.searchDao = searchDao;
    }
//    private SearchLogic searchLogic;
//	
//    public void setSearchLogic(SearchLogic searchLogic) {
//        this.searchLogic = searchLogic;
//    }
    public ActionForward execute(ActionMapping mapping, ActionForm form,
            HttpServletRequest request, HttpServletResponse response) {
		HttpSession session = request.getSession();
		System.out.println("aaaaaa");
		String[] defaultColumns = {"Customer ID", "Customer Name", "Sex", "Birthday", "Address"};
		if (session.getAttribute("selectedColumns") == null) {
			session.setAttribute("selectedColumns", defaultColumns);
		}
		Gson gsons = new Gson();
		String jsons = gsons.toJson(session.getAttribute("selectedColumns"));
		request.setAttribute("t", jsons);

		SearchForm searchForm = (SearchForm) form;
		String customerName = searchForm.getCustomerName();
		String sex = searchForm.getSex();
		String fromBirthday = searchForm.getBirthdayFrom();
		String toBirthday = searchForm.getBirthdayTo();
		ActionMessages errors = new ActionMessages();
		String action = request.getParameter("action");
		
		String sessionCustomerName = (String) session.getAttribute("lastValidCustomerName");
		String sessionSex = (String) session.getAttribute("lastValidSex");
		String sessionFromBirthday = (String) session.getAttribute("lastValidFromBirthday");
		String sessionToBirthday = (String) session.getAttribute("lastValidToBirthday");
		List<CustomerDto> customerList;
		customerList = searchDao.searchCustomers(customerName, sex, fromBirthday, toBirthday);
		Gson gson = new Gson();
		String json = gson.toJson(customerList);
		request.setAttribute("test", json);
		if("upload".equals(action)) {
			return mapping.findForward("settingheader");
		}
		if("edit".equals(action)) {
			return mapping.findForward("edit");
		}
		if ("delete".equals(action)) {
		    String deleteIDs = request.getParameter("customerIds");
		    if (deleteIDs != null && !deleteIDs.isEmpty()) {
		        // Tách các ID bằng dấu phẩy
		        String[] idArray = deleteIDs.split(",");
		        System.out.println(idArray);
		    }
		}
		System.out.println("quay lai 1");
		if("search".equals(action)) {
			System.out.println("search");
		    // Lưu giá trị search hiện tại vào session
		    session.setAttribute("lastValidCustomerName", customerName);
		    session.setAttribute("lastValidSex", sex); 
		    session.setAttribute("lastValidFromBirthday", fromBirthday);
		    session.setAttribute("lastValidToBirthday", toBirthday);

		    // Validate birthday format yyyy/mm/dd
		    if (fromBirthday != null && !fromBirthday.isEmpty() && !fromBirthday.matches("^\\d{4}/\\d{2}/\\d{2}$")) {
		        errors.add("birthdayFrom", new ActionMessage("error.birthday.invalid"));
		        saveErrors(request, errors);
		        
		        // Set lại giá trị vào form trước khi return
		        searchForm.setCustomerName(customerName);
		        searchForm.setSex(sex);
		        searchForm.setBirthdayFrom(fromBirthday); 
		        searchForm.setBirthdayTo(toBirthday);
		        return mapping.findForward("search");
		    }

		    if (toBirthday != null && !toBirthday.isEmpty() && !toBirthday.matches("^\\d{4}/\\d{2}/\\d{2}$")) {
		        errors.add("birthdayTo", new ActionMessage("error.birthday.invalid")); 
		        saveErrors(request, errors);
		        
		        // Set lại giá trị vào form trước khi return
		        searchForm.setCustomerName(customerName);
		        searchForm.setSex(sex);
		        searchForm.setBirthdayFrom(fromBirthday);
		        searchForm.setBirthdayTo(toBirthday);
		        
		        return mapping.findForward("search"); 
		    }

		    customerList = searchDao.searchCustomers(customerName, sex, fromBirthday, toBirthday);
		    String jsons1 = gson.toJson(customerList);
		    request.setAttribute("test", jsons1);
		    return mapping.findForward("search");
		}
		System.out.println("quay lai");
		

		int PAGE_SIZE = 2;
		String pageStr = request.getParameter("page");
		Integer currentPage = 1;
		if (pageStr != null) {
		currentPage = Integer.parseInt(pageStr);
		}
		if (currentPage == null || currentPage < 1) {
		currentPage = 1;
		}
		
		int startRow = (currentPage - 1) * PAGE_SIZE;
		int totalCustomers = customerList.size();
		int totalPages = (int) Math.ceil((double) totalCustomers / PAGE_SIZE);
		
		List<CustomerDto> paginatedList = customerList.subList(
		Math.min(startRow, totalCustomers),
		Math.min(startRow + PAGE_SIZE, totalCustomers)
		);

		
		session.setAttribute("customerList", paginatedList);
		session.setAttribute("currentPage", currentPage);
		session.setAttribute("totalPages", totalPages);
		
		// Lưu lại các giá trị tìm kiếm để hiển thị trên form
		searchForm.setCustomerName(sessionCustomerName);
		searchForm.setSex(sessionSex);
		searchForm.setBirthdayFrom(sessionFromBirthday);
		searchForm.setBirthdayTo(sessionToBirthday);
		
		return mapping.findForward("search");
		}

} 
```
------------ new day again --------------
```
<%@ taglib uri="http://struts.apache.org/tags-html" prefix="html" %>
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c" %>
<%@ taglib uri="/WEB-INF/struts-bean.tld" prefix="bean" %>
<html>
<head>
    <title>Setting Header</title>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/jqgrid/5.3.1/css/ui.jqgrid.min.css">
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jquery/3.6.0/jquery.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jqgrid/5.3.1/js/jquery.jqgrid.min.js"></script>
    <script>
        $(document).ready(function () {
            function loadColumns() {
                $.ajax({
                    url: '<c:url value="/upload.do?action=save" />',
                    method: 'GET',
                    dataType: 'json',
                    success: function (data) {
                        // Load các cột đã chọn và chưa chọn vào jqGrid
                        $("#selectedGrid").jqGrid('clearGridData').jqGrid('setGridParam', { data: data.selectedColumns }).trigger('reloadGrid');
                        $("#availableGrid").jqGrid('clearGridData').jqGrid('setGridParam', { data: data.availableColumns }).trigger('reloadGrid');
                    }
                });
            }

            // Khởi tạo jqGrid cho cột đã chọn
            $("#selectedGrid").jqGrid({
                datatype: 'local',
                colNames: ['Selected Columns'],
                colModel: [{ name: 'column', index: 'column', width: 200 }],
                height: 200,
                viewrecords: true,
                caption: "Selected Columns",
                onSelectRow: toggleMoveButtons
            });

            // Khởi tạo jqGrid cho cột chưa chọn
            $("#availableGrid").jqGrid({
                datatype: 'local',
                colNames: ['Available Columns'],
                colModel: [{ name: 'column', index: 'column', width: 200 }],
                height: 200,
                viewrecords: true,
                caption: "Available Columns",
                onSelectRow: toggleMoveButtons
            });

            // Nạp dữ liệu ban đầu
            loadColumns();

            // Các chức năng di chuyển cột
            $("#moveRightButton").click(function () {
                moveColumn("availableGrid", "selectedGrid");
            });

            $("#moveLeftButton").click(function () {
                moveColumn("selectedGrid", "availableGrid");
            });

            function moveColumn(fromGrid, toGrid) {
                var selRowId = $("#" + fromGrid).jqGrid('getGridParam', 'selrow');
                if (selRowId) {
                    var columnData = $("#" + fromGrid).jqGrid('getRowData', selRowId);
                    $("#" + toGrid).jqGrid('addRowData', selRowId, columnData);
                    $("#" + fromGrid).jqGrid('delRowData', selRowId);
                }
            }

            // Lưu cấu hình cột
            $("#saveButton").click(function () {
                var selectedColumns = $("#selectedGrid").jqGrid('getRowData').map(row => row.column).join(',');
                $.post('<c:url value="/upload.do?action=save" />', { selectedColumns: selectedColumns }, function () {
                    alert("Columns saved successfully!");
                });
            });
        });
    </script>
</head>
<body>
    <h2>Manage Search Screen Columns</h2>
    <html:form>
        <div style="display: flex; gap: 10px;">
            <table id="availableGrid"></table>
            <div style="display: flex; flex-direction: column; gap: 5px; justify-content: center;">
                <button type="button" id="moveRightButton">→</button>
                <button type="button" id="moveLeftButton">←</button>
            </div>
            <table id="selectedGrid"></table>
        </div>
        <br>
        <button type="button" id="saveButton">Save</button>
    </html:form>
</body>
</html>

```
```
import com.google.gson.Gson;
import com.google.gson.JsonObject;
import javax.servlet.http.HttpServletResponse;
// import các thư viện cần thiết

public class UploadAction extends Action {
    private SearchDao searchDao;

    public void setSearchDao(SearchDao searchDao) {
        this.searchDao = searchDao;
    }

    public ActionForward execute(ActionMapping mapping, ActionForm form,
                                 HttpServletRequest request, HttpServletResponse response) throws Exception {
        // Danh sách tất cả các cột
        String[] allColumns = {"Customer ID", "Customer Name", "Sex", "Birthday", "Address", "Email"};
        String selectedColumns = request.getParameter("selectedColumns");
        String action = request.getParameter("action");

        // Khởi tạo JSON để gửi về
        Gson gson = new Gson();
        JsonObject jsonResponse = new JsonObject();

        if ("save".equals(action)) {
            // Xử lý lưu các cột đã chọn
            List<String> selectedList = selectedColumns != null ? Arrays.asList(selectedColumns.split(",")) : new ArrayList<>();
            List<String> availableList = new ArrayList<>();

            // Xác định các cột chưa chọn
            for (String column : allColumns) {
                if (!selectedList.contains(column)) {
                    availableList.add(column);
                }
            }

            // Đặt danh sách cột vào JSON để trả về cho jqGrid
            jsonResponse.add("selectedColumns", gson.toJsonTree(selectedList));
            jsonResponse.add("availableColumns", gson.toJsonTree(availableList));

            // Cấu hình response là JSON
            response.setContentType("application/json");
            response.getWriter().write(gson.toJson(jsonResponse));
            return null; // Không chuyển tiếp tới trang khác
        }

        // Trả về màn hình Setting Header nếu không phải lưu cột
        return mapping.findForward("settingHeader");
    }
}

```
```
package fjs.cs.action;

import java.text.ParseException;
import java.text.SimpleDateFormat;
import java.util.Date;
import java.util.List;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import javax.servlet.http.HttpSession;

import org.apache.struts.action.Action;
import org.apache.struts.action.ActionForm;
import org.apache.struts.action.ActionForward;
import org.apache.struts.action.ActionMapping;
import org.apache.struts.action.ActionMessage;
import org.apache.struts.action.ActionMessages;

import com.google.gson.Gson;

import fjs.cs.dao.SearchDao;
import fjs.cs.dto.CustomerDto;
import fjs.cs.form.SearchForm;

public class SearchAction extends Action {
    
    private SearchDao searchDao;
	
    public void setSearchDao(SearchDao searchDao) {
        this.searchDao = searchDao;
    }
//    private SearchLogic searchLogic;
//	
//    public void setSearchLogic(SearchLogic searchLogic) {
//        this.searchLogic = searchLogic;
//    }
    public ActionForward execute(ActionMapping mapping, ActionForm form,
            HttpServletRequest request, HttpServletResponse response) {
		HttpSession session = request.getSession();
		
		String[] defaultColumns = {"Customer ID", "Customer Name", "Sex", "Birthday", "Address","Email"};
		session.setAttribute("selectedColumns", defaultColumns);
		Gson gsons = new Gson();
		String jsons = gsons.toJson(session.getAttribute("selectedColumns"));
		request.setAttribute("t", jsons);
		System.out.println(jsons);
		SearchForm searchForm = (SearchForm) form;
		String customerName = searchForm.getCustomerName();
		String sex = searchForm.getSex();
		String fromBirthday = searchForm.getBirthdayFrom();
		String toBirthday = searchForm.getBirthdayTo();
		ActionMessages errors = new ActionMessages();
		String action = request.getParameter("action");
		
		String sessionCustomerName = (String) session.getAttribute("lastValidCustomerName");
		String sessionSex = (String) session.getAttribute("lastValidSex");
		String sessionFromBirthday = (String) session.getAttribute("lastValidFromBirthday");
		String sessionToBirthday = (String) session.getAttribute("lastValidToBirthday");
		
		List<CustomerDto> customerList;


		customerList = searchDao.searchCustomers(sessionCustomerName, sessionSex, sessionFromBirthday, sessionToBirthday);
		
		
		int PAGE_SIZE = 2;
		String pageStr = request.getParameter("page");
		Integer currentPage = 1;
		if (pageStr != null) {
		currentPage = Integer.parseInt(pageStr);
		}
		if (currentPage == null || currentPage < 1) {
		currentPage = 1;
		}
		
		int startRow = (currentPage - 1) * PAGE_SIZE;
		int totalCustomers = customerList.size();
		int totalPages = (int) Math.ceil((double) totalCustomers / PAGE_SIZE);
		
		List<CustomerDto> paginatedList = customerList.subList(
		Math.min(startRow, totalCustomers),
		Math.min(startRow + PAGE_SIZE, totalCustomers)
		);
		Gson gson = new Gson();
		String json = gson.toJson(customerList);
		request.setAttribute("test", json);
		
		session.setAttribute("customerList", paginatedList);
		session.setAttribute("currentPage", currentPage);
		session.setAttribute("totalPages", totalPages);
		
		// Lưu lại các giá trị tìm kiếm để hiển thị trên form
		searchForm.setCustomerName(sessionCustomerName);
		searchForm.setSex(sessionSex);
		searchForm.setBirthdayFrom(sessionFromBirthday);
		searchForm.setBirthdayTo(sessionToBirthday);
		
		return mapping.findForward("search");
		}

}
```
```
<%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8"%>
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c"%>
<%@ taglib uri="http://struts.apache.org/tags-html" prefix="html" %>
<%@ taglib uri="/WEB-INF/struts-bean.tld" prefix="bean" %>
<%@ taglib uri="/WEB-INF/struts-logic.tld" prefix="logic" %>
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>Search Customers</title>
    <style type="text/css">
        body {
            margin-left: 20px;
            margin-right: 20px;
            background-color: #bcffff;
        }
        .header {
            border-bottom: 2px solid;
        }
        .header h1 {
            color: red;
        }
        .divider {
            height: 20px;
            width: 100%;
            background-color: #3097ff;
        }
        .search {
            display: flex;
            justify-content: space-around;
            align-items: center;
            padding: 10px;
            margin-top: 20px;
            background-color: #ffff56;
        }
		table {
		    width: 100%;
		    border-collapse: collapse;
		    table-layout :fixed;
		    border :  1px solid green;
		}
		#gbox_customerGrid {
			border :  2px solid green;
			border-collapse: collapse;
		}
		th,
		td {
		    padding: 8px;
		    text-align: left;
		    border-bottom: 1px solid #ddd;
		}
		th {
		    background-color: #f2f2f2;
		}
		tr:first-child > th {
		    background-color: rgb(0, 223, 0);
		    
		}
		tr:nth-child(even) {
		    background-color: #f2f2f2;
		}
    </style>
    
    <!-- Libraries for jqGrid and jQuery -->
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/jqueryui/1.12.1/jquery-ui.min.css">
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/free-jqgrid/4.15.5/css/ui.jqgrid.min.css">
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jquery/3.6.0/jquery.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jqueryui/1.12.1/jquery-ui.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/free-jqgrid/4.15.5/jquery.jqgrid.min.js"></script>
</head>
<body>
    <div class="header">
        <h1>TRAINING</h1>
    </div>
    <div class="container">
<div class="search">
    <input type="text" id="customerName" placeholder="Customer Name">
    <select id="sex">
        <option value="">Any</option>
        <option value="0">Male</option>
        <option value="1">Female</option>
    </select>
    <input type="text" id="birthdayFrom" placeholder="Birthday From (YYYY-MM-DD)">
    <input type="text" id="birthdayTo" placeholder="Birthday To (YYYY-MM-DD)">
    <button id="searchButton">Search</button>
    <button id="deleteButton">Delete Selected</button>
    <button id="firstPageButton">First &lt;&lt;</button>
    <button id="previousPageButton">Previous &lt;</button>
    <button id="nextPageButton">Next &gt;</button>
    <button id="lastPageButton">Last &gt;&gt;</button>
    <button id="settingHeaderButton">Setting Header</button>
</div>
<div></div>
<table id="customerGrid"></table>
<div id="customerPager" style="display:none"></div>
<script type="text/javascript">
    $(document).ready(function () {
    	var selectedColumns = ${t};
    	console.log(selectedColumns);
        $("#customerGrid").jqGrid({
            data: ${test},
            datatype: "local",
            mtype: "POST",
            postData: function() {
                return {
                    customerName: $("#customerName").val(),
                    sex: $("#sex").val(),
                    birthdayFrom: $("#birthdayFrom").val(),
                    birthdayTo: $("#birthdayTo").val()
                };
            },
            colNames: selectedColumns,
            colModel: [
                { name: "customerID", index: "customerID", width: 75, key: true },
                { name: "customerName", index: "customerName", width: 150 },
                { name: "sex", index: "sex", width: 80, formatter: formatSex },
                { name: "birthday", index: "birthday", width: 100 },
                { name: "address", index: "address", width: 200 },
                { name: "email", index: "email", width: 200 }
            ],
            pager: "#customerPager",
            rowNum: 10,
            sortname: "customerID",
            sortorder: "asc",
            viewrecords: true,
            height: "auto",
            autowidth: true,
            multiselect: true, // Enable multiple row selection
             // Enable alternate row colors // Apply the custom class for alternate rows
            jsonReader: {
                repeatitems: false,
                root: "rows",
                page: "page",
                total: "total",
                records: "records"
            },
            gridComplete: function () {
                updatePaginationButtons();
            }
        });
        // Kiểm tra trạng thái các nút phân trang
        function updatePaginationButtons() {
            const currentPage = $("#customerGrid").jqGrid("getGridParam", "page");
            const lastPage = $("#customerGrid").jqGrid("getGridParam", "lastpage");

            // Disable nút Previous nếu ở trang đầu tiên
            $("#previousPageButton").prop("disabled", currentPage <= 1);
            $("#firstPageButton").prop("disabled", currentPage <= 1);
            // Disable nút Next nếu ở trang cuối cùng
            $("#nextPageButton").prop("disabled", currentPage >= lastPage);
            $("#lastPageButton").prop("disabled", currentPage >= lastPage);
        }

        // Gọi hàm updatePaginationButtons sau mỗi lần phân trang
        $("#customerGrid").jqGrid("setGridParam", {
            onPaging: function () {
                setTimeout(updatePaginationButtons, 100); // Delay ngắn để cập nhật sau khi phân trang
            }
        });
        // Search button functionality
        $("#searchButton").click(function () {
            $("#customerGrid").jqGrid('setGridParam', {
                page: 1,
                postData: {
                    customerName: $("#customerName").val(),
                    sex: $("#sex").val(),
                    birthdayFrom: $("#birthdayFrom").val(),
                    birthdayTo: $("#birthdayTo").val()
                }
            }).trigger("reloadGrid");
        });

        // Delete button functionality
        $("#deleteButton").click(function () {
            const selectedIds = $("#customerGrid").jqGrid('getGridParam', 'selarrrow');
            if (selectedIds.length === 0) {
                alert("Please select at least one customer to delete.");
                return;
            }

            if (confirm("Are you sure you want to delete the selected customers?")) {
                $.ajax({
                    url: window.location.pathname.substring(0, window.location.pathname.indexOf("/", 2)) + "/delete.do",
                    type: "POST",
                    data: { customerIds: selectedIds },
                    success: function(response) {
                        alert("Selected customers have been deleted successfully.");
                        $("#customerGrid").trigger("reloadGrid");
                    },
                    error: function() {
                        alert("An error occurred while deleting the customers.");
                    }
                });
            }
            
        });

        // Pagination controls
        $("#firstPageButton").click(function () {
            $("#customerGrid").jqGrid("setGridParam", { page: 1 }).trigger("reloadGrid");
        });

        $("#previousPageButton").click(function () {
            const currentPage = $("#customerGrid").jqGrid("getGridParam", "page");
            if (currentPage > 1) {
                $("#customerGrid").jqGrid("setGridParam", { page: currentPage - 1 }).trigger("reloadGrid");
            }
        });

        $("#nextPageButton").click(function () {
            const currentPage = $("#customerGrid").jqGrid("getGridParam", "page");
            const lastPage = $("#customerGrid").jqGrid("getGridParam", "lastpage");
            if (currentPage < lastPage) {
                $("#customerGrid").jqGrid("setGridParam", { page: currentPage + 1 }).trigger("reloadGrid");
            }
        });

        $("#lastPageButton").click(function () {
            const lastPage = $("#customerGrid").jqGrid("getGridParam", "lastpage");
            $("#customerGrid").jqGrid("setGridParam", { page: lastPage }).trigger("reloadGrid");
        });

        // Formatter for Sex column
        function formatSex(cellValue) {
            return cellValue === "0" ? "Male" : "Female";
        }
        $("#settingHeaderButton").click(function () {
            window.location.href = window.location.pathname.substring(0, window.location.pathname.indexOf("/", 2)) + "/upload.do";
        });
        console.log("${selectedColumns}")
    });
</script>


    </div>
</body>
</html>

```
------ NEW DAY

```
<%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8"%>
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c"%>
<%@ taglib uri="http://struts.apache.org/tags-html" prefix="html" %>
<%@ taglib uri="/WEB-INF/struts-bean.tld" prefix="bean" %>
<%@ taglib uri="/WEB-INF/struts-logic.tld" prefix="logic" %>
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>Search Customers</title>
    <style type="text/css">
        body {
            margin-left: 20px;
            margin-right: 20px;
            background-color: #bcffff;
        }
        .header {
            border-bottom: 2px solid;
        }
        .header h1 {
            color: red;
        }
        .divider {
            height: 20px;
            width: 100%;
            background-color: #3097ff;
        }
        .search {
            display: flex;
            justify-content: space-around;
            align-items: center;
            padding: 10px;
            margin-top: 20px;
            background-color: #ffff56;
        }
    </style>
    
    <!-- Libraries for jqGrid and jQuery -->
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/jqueryui/1.12.1/jquery-ui.min.css">
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/free-jqgrid/4.15.5/css/ui.jqgrid.min.css">
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jquery/3.6.0/jquery.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jqueryui/1.12.1/jquery-ui.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/free-jqgrid/4.15.5/jquery.jqgrid.min.js"></script>
</head>
<body>
    <div class="header">
        <h1>TRAINING</h1>
    </div>
    <div class="container">
        <div class="search">
		    <input type="text" id="customerName" placeholder="Customer Name">
		    <select id="sex">
		        <option value="">Any</option>
		        <option value="0">Male</option>
		        <option value="1">Female</option>
		    </select>
		    <input type="text" id="birthdayFrom" placeholder="Birthday From (YYYY-MM-DD)">
		    <input type="text" id="birthdayTo" placeholder="Birthday To (YYYY-MM-DD)">
		    <button id="searchButton">Search</button>
		    <button id="deleteButton">Delete Selected</button>
		</div>
        <table id="customerGrid"></table>
        <div id="customerPager"></div>

<script type="text/javascript">
    $(document).ready(function () {
        $("#customerGrid").jqGrid({
            url: window.location.pathname.substring(0, window.location.pathname.indexOf("/", 2)) + "/search.do",
            datatype: "json",
            mtype: "POST",
            postData: function() {
                return {
                    customerName: $("#customerName").val(),
                    sex: $("#sex").val(),
                    birthdayFrom: $("#birthdayFrom").val(),
                    birthdayTo: $("#birthdayTo").val()
                };
            },
            colNames: ["Select", "Customer ID", "Customer Name", "Sex", "Birthday", "Address", "Email"],
            colModel: [
                { name: "select", index: "select", width: 50, align: "center", formatter: "checkbox", formatoptions: { disabled: false }},
                { name: "customerID", index: "customerID", width: 75, key: true },
                { name: "customerName", index: "customerName", width: 150 },
                { name: "sex", index: "sex", width: 80, formatter: formatSex },
                { name: "birthday", index: "birthday", width: 100 },
                { name: "address", index: "address", width: 200 },
                { name: "email", index: "email", width: 200 }
            ],
            pager: "#customerPager",
            rowNum: 10,
            rowList: [10, 20, 30],
            sortname: "customerID",
            sortorder: "asc",
            viewrecords: true,
            caption: "Customer List",
            height: "auto",
            autowidth: true,
            multiselect: true, // Enable multiple row selection
            jsonReader: {
                repeatitems: false,
                root: "rows",
                page: "page",
                total: "total",
                records: "records"
            }
        });

        // Search button functionality
        $("#searchButton").click(function () {
            $("#customerGrid").jqGrid('setGridParam', {
                page: 1,
                postData: {
                    customerName: $("#customerName").val(),
                    sex: $("#sex").val(),
                    birthdayFrom: $("#birthdayFrom").val(),
                    birthdayTo: $("#birthdayTo").val()
                }
            }).trigger("reloadGrid");
        });

        // Delete button functionality
        $("#deleteButton").click(function () {
            const selectedIds = $("#customerGrid").jqGrid('getGridParam', 'selarrrow');
            if (selectedIds.length === 0) {
                alert("Please select at least one customer to delete.");
                return;
            }

            if (confirm("Are you sure you want to delete the selected customers?")) {
                $.ajax({
                    url: window.location.pathname.substring(0, window.location.pathname.indexOf("/", 2)) + "/delete.do",
                    type: "POST",
                    data: { customerIds: selectedIds },
                    success: function(response) {
                        alert("Selected customers have been deleted successfully.");
                        $("#customerGrid").trigger("reloadGrid");
                    },
                    error: function() {
                        alert("An error occurred while deleting the customers.");
                    }
                });
            }
        });

        // Formatter for Sex column
        function formatSex(cellValue) {
            return cellValue === "0" ? "Male" : "Female";
        }
    });
</script>

    </div>
</body>
</html>

```
```
<%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8"%>
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c"%>
<%@ taglib uri="http://struts.apache.org/tags-html" prefix="html" %>
<%@ taglib uri="/WEB-INF/struts-bean.tld" prefix="bean" %>
<%@ taglib uri="/WEB-INF/struts-logic.tld" prefix="logic" %>
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>Search Customers</title>
    <style type="text/css">
        body {
            margin-left: 20px;
            margin-right: 20px;
            background-color: #bcffff;
        }
        .header {
            border-bottom: 2px solid;
        }
        .header h1 {
            color: red;
        }
        .divider {
            height: 20px;
            width: 100%;
            background-color: #3097ff;
        }
        .search {
            display: flex;
            justify-content: space-around;
            align-items: center;
            padding: 10px;
            margin-top: 20px;
            background-color: #ffff56;
        }
    </style>
    
    <!-- Thư viện jqGrid và jQuery -->
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/jqueryui/1.12.1/jquery-ui.min.css">
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/free-jqgrid/4.15.5/css/ui.jqgrid.min.css">
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jquery/3.6.0/jquery.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jqueryui/1.12.1/jquery-ui.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/free-jqgrid/4.15.5/jquery.jqgrid.min.js"></script>
</head>
<body>
    <div class="header">
        <h1>TRAINING</h1>
    </div>
    <div class="container">
        <!-- jqGrid Table -->
        <div class="search">
		    <input type="text" id="customerName" placeholder="Customer Name">
		    <select id="sex">
		        <option value="">Any</option>
		        <option value="0">Male</option>
		        <option value="1">Female</option>
		    </select>
		    <input type="text" id="birthdayFrom" placeholder="Birthday From (YYYY-MM-DD)">
		    <input type="text" id="birthdayTo" placeholder="Birthday To (YYYY-MM-DD)">
		    <button id="searchButton">Search</button>
		</div>
        <table id="customerGrid"></table>
        <div id="customerPager"></div>

<script type="text/javascript">
    $(document).ready(function () {
        $("#customerGrid").jqGrid({
            url: window.location.pathname.substring(0, window.location.pathname.indexOf("/", 2)) + "/search.do",
            datatype: "json",
            mtype: "POST",
            postData: function() {
                return {
                    customerName: $("#customerName").val(),
                    sex: $("#sex").val(),
                    birthdayFrom: $("#birthdayFrom").val(),
                    birthdayTo: $("#birthdayTo").val()
                };
            },
            colNames: ["Customer ID", "Customer Name", "Sex", "Birthday", "Address", "Email"],
            colModel: [
                { name: "customerID", index: "customerID", width: 75, key: true },
                { name: "customerName", index: "customerName", width: 150 },
                { name: "sex", index: "sex", width: 80, formatter: formatSex },
                { name: "birthday", index: "birthday", width: 100 },
                { name: "address", index: "address", width: 200 },
                { name: "email", index: "email", width: 200 }
            ],
            pager: "#customerPager",
            rowNum: 10,
            rowList: [10, 20, 30],
            sortname: "customerID",
            sortorder: "asc",
            viewrecords: true,
            caption: "Customer List",
            height: "auto",
            autowidth: true,
            jsonReader: {
                repeatitems: false,
                root: "rows",
                page: "page",
                total: "total",
                records: "records"
            }
        });

        // Trigger search on input change or button click
        $("#searchButton").click(function () {
            $("#customerGrid").jqGrid('setGridParam', {
                page: 1,
                postData: {
                    customerName: $("#customerName").val(),
                    sex: $("#sex").val(),
                    birthdayFrom: $("#birthdayFrom").val(),
                    birthdayTo: $("#birthdayTo").val()
                }
            }).trigger("reloadGrid");
        });

        // Formatter for Sex column
        function formatSex(cellValue) {
            return cellValue === "0" ? "Male" : "Female";
        }
    });
</script>

    </div>
</body>
</html>

```
```
<%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8"%>
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c"%>
<%@ taglib uri="http://struts.apache.org/tags-html" prefix="html" %>
<%@ taglib uri="/WEB-INF/struts-bean.tld" prefix="bean" %>
<%@ taglib uri="/WEB-INF/struts-logic.tld" prefix="logic" %>

<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>Search Customers</title>
<style type="text/css">
    /* Add styles for jqGrid */
    .ui-jqgrid .ui-jqgrid-btable {
        table-layout: auto;
    }
    .ui-jqgrid-sortable {
        cursor: pointer;
    }
    .ui-jqgrid-pager {
        height: 25px;
    }
</style>

<!-- Include jQuery and jqGrid library -->
<script src="https://code.jquery.com/jquery-3.6.0.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/jqGrid/4.6.0/js/jquery.jqGrid.min.js"></script>
<link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/jqGrid/4.6.0/css/ui.jqgrid.min.css">
</head>
<body>
    <div class="header">
        <h1>TRAINING</h1>
    </div>
    <div class="container">
        <div class="welcome_user">
            <p>Welcome, <%=session.getAttribute("userName")%></p>
            <a href="#">Log out</a>
        </div>
        <html:errors/>
        <div class="divider"></div>
        <html:form action="/search" method="post">
        <div class="search">
            <!-- Search form controls -->
        </div>
        <div id="grid-container">
            <table id="customerGrid"></table>
            <div id="pager"></div>
        </div>
        <div style="padding-block: 20px; display: flex; gap: 20px">
            <button id="addNew" type="button">Add New</button>
            <button id="deleteButton" type="submit" name="action" value="delete">Delete</button>
        </div>
        </html:form>
    </div>

    <script>
        $(document).ready(function () {
            // Initialize jqGrid
            $("#customerGrid").jqGrid({
                url: "<c:url value='/search?action=data'/>",
                datatype: "json",
                colModel: [
                    { label: 'Customer ID', name: 'customerID', width: 100, key: true },
                    { label: 'Customer Name', name: 'customerName', width: 200 },
                    { label: 'Sex', name: 'sex', width: 100,
                        formatter: function (cellvalue, options, rowObject) {
                            return cellvalue === '0' ? 'Male' : 'Female';
                        }
                    },
                    { label: 'Birthday', name: 'birthday', width: 150 },
                    { label: 'Address', name: 'address', width: 300 },
                    { label: 'Email', name: 'email', width: 200 }
                ],
                pager: '#pager',
                rowNum: 10,
                rowList: [10, 20, 30],
                sortname: 'customerID',
                sortorder: "asc",
                viewrecords: true,
                gridview: true,
                caption: "Customer List"
            });
        });
    </script>
</body>
</html>
```
```
<%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8"%>
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c"%>
<%@ taglib uri="http://struts.apache.org/tags-html" prefix="html" %>
<%@ taglib uri="/WEB-INF/struts-bean.tld" prefix="bean" %>
<%@ taglib uri="/WEB-INF/struts-logic.tld" prefix="logic" %>
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>Search Customers</title>
    <style type="text/css">
        body {
            margin-left: 20px;
            margin-right: 20px;
            background-color: #bcffff;
        }
        .header {
            border-bottom: 2px solid;
        }
        .header h1 {
            color: red;
        }
        .divider {
            height: 20px;
            width: 100%;
            background-color: #3097ff;
        }
        .search {
            display: flex;
            justify-content: space-around;
            align-items: center;
            padding: 10px;
            margin-top: 20px;
            background-color: #ffff56;
        }
    </style>
    
    <!-- Thư viện jqGrid và jQuery -->
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/jqueryui/1.12.1/jquery-ui.min.css">
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/free-jqgrid/4.15.5/css/ui.jqgrid.min.css">
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jquery/3.6.0/jquery.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jqueryui/1.12.1/jquery-ui.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/free-jqgrid/4.15.5/jquery.jqgrid.min.js"></script>
</head>
<body>
    <div class="header">
        <h1>TRAINING</h1>
    </div>
    <div class="container">
        <div class="welcome_user">
            <p>Welcome, <%=session.getAttribute("userName")%></p>
            <a href="#">Log out</a>
        </div>
        <html:errors/>
        <div class="divider"></div>

        <!-- Form Tìm kiếm -->
        <div class="search">
            <div>
                <label for="search">Customer Name:</label>
                <input type="text" id="customerName" name="customerName" />
            </div>
            <div>
                <label for="search">Sex:</label>
                <select id="sex" name="sex">
                    <option value=" "> </option>
                    <option value="0">Male</option>
                    <option value="1">Female</option>
                </select>
            </div>
            <div style="display: flex; justify-content: center; align-items: center;">
                <label for="search">Birthday:</label>
                <input type="text" id="birthdayFrom" name="birthdayFrom" />
                <p>~</p>
                <input type="text" id="birthdayTo" name="birthdayTo" />
            </div>
            <button type="button" onclick="reloadGrid()">Search</button>
            <button type="button" onclick="importCSV()">Import CSV</button>
            <button type="button" onclick="exportCSV()">Export CSV</button>
            <button type="button" onclick="openSettings()">SettingHeader</button>
        </div>

        <!-- jqGrid Table -->
        <table id="customerGrid"></table>
        <div id="customerPager"></div>

        <script type="text/javascript">
            $(document).ready(function () {
                // Cấu hình jqGrid
                $("#customerGrid").jqGrid({
                	url: window.location.pathname.substring(0, window.location.pathname.indexOf("/",2)) + "/search",
                    datatype: "json",
                    mtype: "POST",
                    postData: {
                        customerName: function () { return $("#customerName").val(); },
                        sex: function () { return $("#sex").val(); },
                        birthdayFrom: function () { return $("#birthdayFrom").val(); },
                        birthdayTo: function () { return $("#birthdayTo").val(); }
                    },
                    colNames: ["Customer ID", "Customer Name", "Sex", "Birthday", "Address", "Email"],
                    colModel: [
                        { name: "customerID", index: "customerID", width: 75, key: true },
                        { name: "customerName", index: "customerName", width: 150 },
                        { name: "sex", index: "sex", width: 80, formatter: formatSex },
                        { name: "birthday", index: "birthday", width: 100 },
                        { name: "address", index: "address", width: 200 },
                        { name: "email", index: "email", width: 200 }
                    ],
                    pager: "#customerPager",
                    rowNum: 10,
                    rowList: [10, 20, 30],
                    sortname: "customerID",
                    sortorder: "asc",
                    viewrecords: true,
                    caption: "Customer List",
                    height: "auto",
                    autowidth: true,
                    jsonReader: {
                        repeatitems: false,
                        root: "rows",
                        page: "page",
                        total: "total",
                        records: "records"
                    }
                });

                // Hàm định dạng cột Sex
                function formatSex(cellValue) {
                    return cellValue === "0" ? "Male" : "Female";
                }

                // Hàm tải lại lưới dữ liệu
                window.reloadGrid = function() {
                    $("#customerGrid").jqGrid("setGridParam", { page: 1 }).trigger("reloadGrid");
                };

                // Các hàm Import và Export CSV, mở màn hình Settings
                window.importCSV = function() {
                    // Viết logic import CSV tại đây
                    alert("Import CSV function called!");
                };

                window.exportCSV = function() {
                    // Viết logic export CSV tại đây
                    alert("Export CSV function called!");
                };

                window.openSettings = function() {
                    // Điều hướng đến trang cài đặt header
                    window.location.href = "${pageContext.request.contextPath}/settingHeader.jsp";
                };
            });
        </script>
    </div>
</body>
</html>

```
```
hibernate 4.
import org.hibernate.Session;
import org.hibernate.Transaction;
import org.hibernate.query.Query;
import java.util.List;

public List<CustomerDto> getAllCustomers() {
    Session session = HibernateUtil.getSessionFactory().openSession();
    Transaction transaction = null;
    List<CustomerDto> customerList = null;

    try {
        transaction = session.beginTransaction();
        Query<CustomerDto> query = session.createQuery("FROM CustomerDto", CustomerDto.class);
        customerList = query.list();
        transaction.commit();
    } catch (Exception e) {
        if (transaction != null) {
            transaction.rollback();
        }
        e.printStackTrace();
    } finally {
        session.close();
    }
    
    return customerList;
}

```
```
import org.hibernate.Session;
import org.hibernate.Transaction;
import org.hibernate.query.Query;
import java.util.List;

public List<CustomerDto> searchCustomers(String customerName, String sex, String fromBirthday, String toBirthday) {
    Session session = HibernateUtil.getSessionFactory().openSession();
    Transaction transaction = null;
    List<CustomerDto> customerList = null;

    try {
        transaction = session.beginTransaction();
        String hql = "FROM CustomerDto c WHERE 1=1"; // Bắt đầu với một điều kiện đúng

        if (customerName != null && !customerName.isEmpty()) {
            hql += " AND c.customerName LIKE :customerName";
        }
        if (sex != null && !sex.isEmpty()) {
            hql += " AND c.sex = :sex";
        }
        if (fromBirthday != null && !fromBirthday.isEmpty()) {
            hql += " AND c.birthday >= :fromBirthday";
        }
        if (toBirthday != null && !toBirthday.isEmpty()) {
            hql += " AND c.birthday <= :toBirthday";
        }

        Query<CustomerDto> query = session.createQuery(hql, CustomerDto.class);

        if (customerName != null && !customerName.isEmpty()) {
            query.setParameter("customerName", "%" + customerName + "%");
        }
        if (sex != null && !sex.isEmpty()) {
            query.setParameter("sex", sex);
        }
        if (fromBirthday != null && !fromBirthday.isEmpty()) {
            query.setParameter("fromBirthday", fromBirthday);
        }
        if (toBirthday != null && !toBirthday.isEmpty()) {
            query.setParameter("toBirthday", toBirthday);
        }

        customerList = query.list();
        transaction.commit();
    } catch (Exception e) {
        if (transaction != null) {
            transaction.rollback();
        }
        e.printStackTrace();
    } finally {
        session.close();
    }
    
    return customerList;
}

```
```
package fjs.cs.dao;

import java.util.ArrayList;
import java.util.List;

import org.hibernate.Criteria;
import org.hibernate.Query;
import org.hibernate.Session;
import org.hibernate.SessionFactory;
import org.hibernate.Transaction;

import fjs.cs.dto.CustomerDto;
import fjs.cs.model.Customer;

public class SearchDao {
	
	private SessionFactory sessionFactory;

	public void setSessionFactory(SessionFactory sessionFactory) {
	   this.sessionFactory = sessionFactory;
	}
    public List<CustomerDto> getAllCustomers() {
        Session session = sessionFactory.openSession();
        String hql = "SELECT c.customerID, c.customerName, c.sex, c.birthday, c.address " +
                     "FROM Customer c WHERE c.delete_ymd IS NULL";
        
        List<?> results = session.createQuery(hql).list();
        session.close();

        List<CustomerDto> customers = new ArrayList<>();
        for (Object result : results) {
            Object[] row = (Object[]) result;
            CustomerDto dto = new CustomerDto();
            dto.setCustomerID((int) row[0]);
            dto.setCustomerName((String) row[1]);
            dto.setSex((String) row[2]);
            dto.setBirthday((String) row[3]);
            dto.setAddress((String) row[4]);
            customers.add(dto);
        }

        return customers;
    }
	
    public List<CustomerDto> searchCustomers(String customerName, String sex, String birthdayFrom, String birthdayTo) {
        Session session = sessionFactory.openSession();
        
        try {
            // Tạo câu truy vấn HQL
            StringBuilder hql = new StringBuilder("SELECT c.customerID, c.customerName, c.sex, c.birthday, c.address FROM Customer c WHERE c.delete_ymd IS NULL");
            
            // Thêm điều kiện tìm kiếm
            if (customerName != null && !customerName.trim().isEmpty()) {
                hql.append(" AND c.customerName LIKE :customerName");
            }
            if (sex != null && !sex.trim().isEmpty()) {
                hql.append(" AND c.sex = :sex");
            }
            if (birthdayFrom != null && !birthdayFrom.trim().isEmpty()) {
                hql.append(" AND c.birthday >= :birthdayFrom");
            }
            if (birthdayTo != null && !birthdayTo.trim().isEmpty()) {
                hql.append(" AND c.birthday <= :birthdayTo");
            }

            // Tạo query
            Query query = session.createQuery(hql.toString());

            // Gán giá trị cho các tham số
            if (customerName != null && !customerName.trim().isEmpty()) {
                query.setParameter("customerName", "%" + customerName + "%");
            }
            if (sex != null && !sex.trim().isEmpty()) {
                query.setParameter("sex", sex);
            }
            if (birthdayFrom != null && !birthdayFrom.trim().isEmpty()) {
                query.setParameter("birthdayFrom", birthdayFrom);
            }
            if (birthdayTo != null && !birthdayTo.trim().isEmpty()) {
                query.setParameter("birthdayTo", birthdayTo);
            }

            // Lấy danh sách kết quả trả về
            List<?> results = query.list();
            List<CustomerDto> customerDtos = new ArrayList<>();

            // Xử lý kết quả trả về
            for (Object result : results) {
                Object[] row = (Object[]) result;
                CustomerDto dto = new CustomerDto(
                    (Integer) row[0],   // customerID
                    (String) row[1],    // customerName
                    (String) row[2],    // sex
                    (String) row[3],    // birthday
                    (String) row[4]     // address
                );
                customerDtos.add(dto);
            }
            
            return customerDtos;
        } finally {
            session.close();
        }
    }

	public void addCustomer(CustomerDto customerDto) {
	    Transaction transaction = null;
	    Session session = sessionFactory.openSession();
	    
	    try {
	        // Bắt đầu transaction
	        transaction = session.beginTransaction();

	        // Tạo đối tượng Customer từ CustomerDto
	        Customer customer = new Customer();
	        customer.setCustomerID(customerDto.getCustomerID()); // Nếu ID tự động sinh, bạn có thể bỏ qua dòng này
	        customer.setCustomerName(customerDto.getCustomerName());
	        customer.setSex(customerDto.getSex());
	        customer.setBirthday(customerDto.getBirthday());
	        customer.setAddress(customerDto.getAddress());
	        customer.setEmail(customerDto.getEmail()); // Nếu có email trong Dto
	        
	        // Lưu đối tượng Customer vào database
	        session.save(customer);
	        
	        // Commit transaction
	        transaction.commit();
	        
	    } catch (Exception e) {
	        if (transaction != null) {
	            transaction.rollback(); // Rollback nếu có lỗi
	        }
	        e.printStackTrace();
	    } finally {
	        session.close(); // Đóng session sau khi hoàn thành
	    }
	}
    public boolean isCustomerExists(int customerId) {
    	Session session = sessionFactory.openSession();
        boolean exists = false;

        try {
            // HQL query để kiểm tra sự tồn tại của customerId
            String hql = "SELECT count(c.customerID) FROM Customer c WHERE c.customerID = :customerID AND c.delete_ymd IS NULL";
            Query query = session.createQuery(hql); // Không sử dụng kiểu tham số hóa ở đây
            query.setParameter("customerID", customerId);

            Long count = (Long) query.uniqueResult(); // Lấy kết quả duy nhất
            exists = (count != null && count > 0); // Nếu có kết quả và > 0, thì customerId tồn tại
        } catch (Exception e) {
            e.printStackTrace(); // Xử lý lỗi
        } finally {
            session.close(); // Đóng session
        }
        return exists;
    }
    public void editCustomer(CustomerDto customerDto) {
        Transaction transaction = null;
        Session session = sessionFactory.openSession();

        try {
            // Bắt đầu transaction
            transaction = session.beginTransaction();

            // Tìm đối tượng Customer từ database theo customerID
            Customer customer = (Customer) session.get(Customer.class, customerDto.getCustomerID());

            if (customer != null) {
                // Cập nhật thông tin từ CustomerDto
                customer.setCustomerName(customerDto.getCustomerName());
                customer.setSex(customerDto.getSex());
                customer.setBirthday(customerDto.getBirthday());
                customer.setAddress(customerDto.getAddress());
                customer.setEmail(customerDto.getEmail()); // Nếu có email trong Dto

                // Lưu đối tượng Customer đã được cập nhật vào database
                session.update(customer);

                // Commit transaction
                transaction.commit();
            } else {
                // Nếu không tìm thấy customer, có thể xử lý logic ở đây (ví dụ: throw exception)
                System.out.println("Customer with ID " + customerDto.getCustomerID() + " not found.");
            }

        } catch (Exception e) {
            if (transaction != null) {
                transaction.rollback(); // Rollback nếu có lỗi
            }
            e.printStackTrace();
        } finally {
            session.close(); // Đóng session sau khi hoàn thành
        }
    }
}

```
```
package fjs.cs.action;

import java.text.ParseException;
import java.text.SimpleDateFormat;
import java.util.Date;
import java.util.List;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import javax.servlet.http.HttpSession;

import org.apache.struts.action.Action;
import org.apache.struts.action.ActionForm;
import org.apache.struts.action.ActionForward;
import org.apache.struts.action.ActionMapping;
import org.apache.struts.action.ActionMessage;
import org.apache.struts.action.ActionMessages;

import fjs.cs.dao.SearchDao;
import fjs.cs.dto.CustomerDto;
import fjs.cs.form.SearchForm;

public class SearchAction extends Action {
    
    private SearchDao searchDao;
	
    public void setSearchDao(SearchDao searchDao) {
        this.searchDao = searchDao;
    }
//    private SearchLogic searchLogic;
//	
//    public void setSearchLogic(SearchLogic searchLogic) {
//        this.searchLogic = searchLogic;
//    }
    public ActionForward execute(ActionMapping mapping, ActionForm form,
            HttpServletRequest request, HttpServletResponse response) {
		HttpSession session = request.getSession();
		
		String[] defaultColumns = {"Customer ID", "Customer Name", "Sex", "Birthday", "Address"};
		if (session.getAttribute("selectedColumns") == null) {
		session.setAttribute("selectedColumns", defaultColumns);
		}
		
		SearchForm searchForm = (SearchForm) form;
		String customerName = searchForm.getCustomerName();
		String sex = searchForm.getSex();
		String fromBirthday = searchForm.getBirthdayFrom();
		String toBirthday = searchForm.getBirthdayTo();
		ActionMessages errors = new ActionMessages();
		String action = request.getParameter("action");
		
		String sessionCustomerName = (String) session.getAttribute("lastValidCustomerName");
		String sessionSex = (String) session.getAttribute("lastValidSex");
		String sessionFromBirthday = (String) session.getAttribute("lastValidFromBirthday");
		String sessionToBirthday = (String) session.getAttribute("lastValidToBirthday");
		
		List<CustomerDto> customerList;
		if("export".equals(action)) {
    	    customerList = searchDao.searchCustomers(customerName, sex, fromBirthday, toBirthday);
    	    
    	    // Tạo file CSV
    	    StringBuilder csvData = new StringBuilder();
    	    
    	    // Tiêu đề của các cột
    	    csvData.append("\"Customer ID\",\"Name\",\"Sex\",\"Birthday\",\"Address\",\"Email\"\n");
    	    
    	    // Thêm dữ liệu khách hàng vào CSV
    	    for (CustomerDto customer : customerList) {
    	        csvData.append("\"").append(customer.getCustomerID()).append("\",")
    	               .append("\"").append(customer.getCustomerName()).append("\",")
    	               .append("\"").append(customer.getSex() == "0" ? "Female" : "Male").append("\",")
    	               .append("\"").append(customer.getBirthday()).append("\",")
    	               .append("\"").append(customer.getAddress()).append("\",")
    	               .append("\"").append(customer.getEmail()).append("\"\n");
    	    }
    	    
    	    // Đường dẫn đến nơi lưu file CSV
    	    String filePath = request.getServletContext().getRealPath("/") + "csv/Tests.csv";
    	    
    	    try {
    	        // Ghi dữ liệu CSV vào file trên ổ đĩa
    	        java.nio.file.Files.write(java.nio.file.Paths.get(filePath), csvData.toString().getBytes());
    	        System.out.println("File exported successfully to " + filePath);
    	    } catch (Exception e) {
    	        e.printStackTrace();
    	    }
		}
		if ("Search".equals(action)) {
		String datePattern = "^\\d{4}/\\d{2}/\\d{2}$";
		if (!fromBirthday.isEmpty() && !fromBirthday.matches(datePattern)) {
		errors.add("birthday", new ActionMessage("error.birthday.invalid"));
		saveErrors(request, errors);
		
		searchForm.setBirthdayFrom(fromBirthday);
		customerList = searchDao.searchCustomers(sessionCustomerName, sessionSex, sessionFromBirthday, sessionToBirthday);
		} else {
		session.setAttribute("lastValidCustomerName", customerName);
		session.setAttribute("lastValidSex", sex);
		session.setAttribute("lastValidFromBirthday", fromBirthday);
		session.setAttribute("lastValidToBirthday", toBirthday);
		
		customerList = searchDao.searchCustomers(customerName, sex, fromBirthday, toBirthday);
		}
		} else {
		customerList = searchDao.searchCustomers(sessionCustomerName, sessionSex, sessionFromBirthday, sessionToBirthday);
		}
		
		int PAGE_SIZE = 2;
		String pageStr = request.getParameter("page");
		Integer currentPage = 1;
		if (pageStr != null) {
		currentPage = Integer.parseInt(pageStr);
		}
		if (currentPage == null || currentPage < 1) {
		currentPage = 1;
		}
		
		int startRow = (currentPage - 1) * PAGE_SIZE;
		int totalCustomers = customerList.size();
		int totalPages = (int) Math.ceil((double) totalCustomers / PAGE_SIZE);
		
		List<CustomerDto> paginatedList = customerList.subList(
		Math.min(startRow, totalCustomers),
		Math.min(startRow + PAGE_SIZE, totalCustomers)
		);
		
		session.setAttribute("customerList", paginatedList);
		session.setAttribute("currentPage", currentPage);
		session.setAttribute("totalPages", totalPages);
		
		// Lưu lại các giá trị tìm kiếm để hiển thị trên form
		searchForm.setCustomerName(sessionCustomerName);
		searchForm.setSex(sessionSex);
		searchForm.setBirthdayFrom(sessionFromBirthday);
		searchForm.setBirthdayTo(sessionToBirthday);
		
		return mapping.findForward("search");
		}

}
```
```
if("export".equals(action)) {
    customerList = searchDao.searchCustomers(customerName, sex, fromBirthday, toBirthday);

    // Tạo file CSV
    StringBuilder csvData = new StringBuilder();

    // Tiêu đề của các cột
    csvData.append("\"Customer ID\",\"Name\",\"Sex\",\"Birthday\",\"Address\",\"Email\"\n");

    // Thêm dữ liệu khách hàng vào CSV
    for (CustomerDto customer : customerList) {
        csvData.append("\"").append(customer.getCustomerID()).append("\",")
            .append("\"").append(customer.getCustomerName()).append("\",")
            .append("\"").append(customer.getSex() == "0" ? "Female" : "Male").append("\",")
            .append("\"").append(customer.getBirthday()).append("\",")
            .append("\"").append(customer.getAddress()).append("\",")
            .append("\"").append(customer.getEmail()).append("\"\n");
    }

    // Lấy đường dẫn tương đối của thư mục CSV
    String csvDirPath = "csv";
    String fileName = "Test.csv";
    String filePath = csvDirPath + File.separator + fileName;

    try {
        // Tạo thư mục csv nếu chưa tồn tại
        new File(csvDirPath).mkdirs();

        // Ghi dữ liệu CSV vào file
        java.nio.file.Files.write(java.nio.file.Paths.get(filePath), csvData.toString().getBytes());
        System.out.println("File exported successfully to " + filePath);
    } catch (Exception e) {
        e.printStackTrace();
    }
}
```
```
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c"%>
<%@ taglib uri="http://struts.apache.org/tags-html" prefix="html" %>
<%@ taglib uri="/WEB-INF/struts-bean.tld" prefix="bean" %>
<%@ taglib uri="/WEB-INF/struts-logic.tld" prefix="logic" %>
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>Search Customers</title>
<style type="text/css">
    body {
        margin-left: 20px;
        margin-right: 20px;
        background-color: #bcffff;
    }
    .header {
        border-bottom: 2px solid;
    }
    .header h1 {
        color: red;
    }
    .welcome_user {
        display: flex;
        align-items: center;
        justify-content: space-between;
    }
    .divider {
        height: 20px;
        width: 100%;
        background-color: #3097ff;
    }
    .search form {
        display: flex;
        justify-content: space-around;
        align-items: center;
        padding: 10px;
        margin-top: 20px;
        background-color: #ffff56;
    }
    .pagination {
        margin-top: 10px;
    }
    .pagination .navigation-left {
        display: flex;
        gap: 5px;
        align-items: center;
    }
    .pagination .navigation-right {
        display: flex;
        gap: 5px;
        margin-left: auto;
        align-items: center;
    }
    table {
        width: 100%;
        border-collapse: collapse;
    }
    th, td {
        padding: 8px;
        text-align: left;
        border-bottom: 1px solid #ddd;
    }
    th {
        background-color: #f2f2f2;
    }
    tr:first-child > th {
        background-color: rgb(0, 223, 0);
    }
    tr:nth-child(even) {
        background-color: #f2f2f2;
    }
</style>
</head>
<body>
    <div class="header">
        <h1>TRAINING</h1>
    </div>
    <div class="container">
        <div class="welcome_user">
            <p>Welcome, <%=session.getAttribute("userName")%></p>
            <a href="#">Log out</a>
        </div>
        <html:errors/>
        <div class="divider"></div>
        <html:form action="/search" method="post">
        <div class="search">
            <div>
                <label for="search">Customer Name:</label>
                <html:text property="customerName" />
            </div>
            <div>
                <label for="search">Sex:</label>
                <html:select property="sex">
                    <html:option value=" "></html:option>
                    <html:option value="0">Male</html:option>
                    <html:option value="1">Female</html:option>
                </html:select>
            </div>
            <div style="display: flex; justify-content: center; align-items: center;">
                <label for="search">Birthday:</label>
                <html:text property="birthdayFrom" styleClass="birthday" />
                <p>~</p>
                <html:text property="birthdayTo" styleClass="birthday" />
            </div>
            <button type="submit" value="Search" name="action">Search</button>
            <button type="submit" value="import" name="action">Import CSV</button>
            <button type="submit" value="SettingHeader" name="action">SettingHeader</button>
        	<button type="submit" value="export" name="action">Export CSV</button>
        </div>
        <div class="pagination">
            <div class="navigation-left">
                <logic:present name="currentPage">
                    <logic:greaterThan name="currentPage" value="1">
                        <button type="submit" name="page" value="1">&laquo;</button>
                    </logic:greaterThan>
                    <logic:lessEqual name="currentPage" value="1">
                        <button type="button" disabled>&laquo;</button>
                    </logic:lessEqual>
                </logic:present>

                <logic:present name="currentPage">
                    <logic:greaterThan name="currentPage" value="1">
                        <button type="submit" name="page" value="${currentPage - 1}">&lt;</button>
                    </logic:greaterThan>
                    <logic:lessEqual name="currentPage" value="1">
                        <button type="button" disabled>&lt;</button>
                    </logic:lessEqual>
                </logic:present>
                <p>Previous</p>
            </div>
            <div class="navigation-right">
                <p>Next</p>
                <logic:present name="currentPage">
                    <logic:lessThan name="currentPage" value="${totalPages}">
                        <button type="submit" name="page" value="${currentPage + 1}">&gt;</button>
                    </logic:lessThan>
                    <logic:greaterEqual name="currentPage" value="${totalPages}">
                        <button type="button" disabled>&gt;</button>
                    </logic:greaterEqual>
                </logic:present>

                <logic:present name="currentPage">
                    <logic:lessThan name="currentPage" value="${totalPages}">
                        <button type="submit" name="page" value="${totalPages}">&raquo;</button>
                    </logic:lessThan>
                    <logic:greaterEqual name="currentPage" value="${totalPages}">
                        <button type="button" disabled>&raquo;</button>
                    </logic:greaterEqual>
                </logic:present>
            </div>
        </div>
        <div class="table">
            <table>
                <tr>
                    <th><input type="checkbox" id="select-all" onclick="selectAllCheckboxes(this)"></th>
                    <logic:iterate id="column" name="selectedColumns">
                        <th><bean:write name="column" /></th>
                    </logic:iterate>
                </tr>
                <logic:iterate id="customer" name="customerList">
                    <tr>
                        <td><input type="checkbox" name="deleteIds" value="${customer.customerID}" onclick="updateSelectAll()"></td>
                        <logic:iterate id="column" name="selectedColumns" scope="session">
                            <td>
                                <logic:equal name="column" value="Customer ID">
                                    <a href="edit?id=${customer.customerID}"><bean:write name="customer" property="customerID" /></a>
                                </logic:equal>
                                <logic:equal name="column" value="Customer Name">
                                    <bean:write name="customer" property="customerName" />
                                </logic:equal>
                                <logic:equal name="column" value="Sex">
                                    <logic:equal name="customer" property="sex" value="0">Male</logic:equal>
                                    <logic:equal name="customer" property="sex" value="1">Female</logic:equal>
                                </logic:equal>
                                <logic:equal name="column" value="Birthday">
                                    <bean:write name="customer" property="birthday" />
                                </logic:equal>
                                <logic:equal name="column" value="Address">
                                    <bean:write name="customer" property="address" />
                                </logic:equal>
                                <logic:equal name="column" value="Email">
                                    <bean:write name="customer" property="email" />
                                </logic:equal>
                            </td>
                        </logic:iterate>
                    </tr>
                </logic:iterate>
            </table>
            <div style="padding-block: 20px; display: flex; gap: 20px">
                <button id="addNew" type="button">Add New</button>
                <button id="deleteButton" type="submit" name="action" value="delete">Delete</button>
            </div>
        </div>
        </html:form>
    </div>
</body>
</html>
```
https://drive.google.com/drive/folders/1bPh22WOHnfjV8E2gzLIvzQr1Bx1uL9nx?usp=drive_link
```
search action
package fjs.cs.action;

import java.text.ParseException;
import java.text.SimpleDateFormat;
import java.util.Date;
import java.util.List;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import javax.servlet.http.HttpSession;

import org.apache.struts.action.Action;
import org.apache.struts.action.ActionForm;
import org.apache.struts.action.ActionForward;
import org.apache.struts.action.ActionMapping;
import org.apache.struts.action.ActionMessage;
import org.apache.struts.action.ActionMessages;

import fjs.cs.dao.SearchDao;
import fjs.cs.dto.CustomerDto;
import fjs.cs.form.SearchForm;

public class SearchAction extends Action {
    
    private SearchDao searchDao;
	
    public void setSearchDao(SearchDao searchDao) {
        this.searchDao = searchDao;
    }
//    private SearchLogic searchLogic;
//	
//    public void setSearchLogic(SearchLogic searchLogic) {
//        this.searchLogic = searchLogic;
//    }
	public ActionForward execute(ActionMapping mapping, ActionForm form,
            HttpServletRequest request, HttpServletResponse response) {
    	HttpSession session = request.getSession();
    	
        String[] defaultColumns = {"Customer ID", "Customer Name", "Sex", "Birthday", "Address"};
        
        // Nếu selectedColumns chưa được lưu trong session thì thiết lập mặc định
        if (session.getAttribute("selectedColumns") == null) {
            session.setAttribute("selectedColumns", defaultColumns);
        } else {
            // Nếu đã có selectedColumns trong session, sử dụng danh sách hiện tại
            String[] selectedColumns = (String[]) session.getAttribute("selectedColumns");
            session.setAttribute("selectedColumns", selectedColumns);
            // Nếu selectedColumns rỗng hoặc không hợp lệ, thiết lập lại mặc định
            if (selectedColumns.length == 0) {
                session.setAttribute("selectedColumns", defaultColumns);
            }
        }
        
//        String[] selected = (String[]) session.getAttribute("selectedColumns");
//        System.out.println("con day " +selected);

    	SearchForm searchForm = (SearchForm) form;
    	String customerName = searchForm.getCustomerName();
        String sex = searchForm.getSex();
        String fromBirthday = searchForm.getBirthdayFrom();
        String toBirthday = searchForm.getBirthdayTo();
        ActionMessages errors = new ActionMessages();
        
        String action = request.getParameter("action");
    	List<CustomerDto> customerList;

    	if ("export".equals(action)) {
    	    customerList = searchDao.searchCustomers(customerName, sex, fromBirthday, toBirthday);
    	    
    	    // Tạo file CSV
    	    StringBuilder csvData = new StringBuilder();
    	    
    	    // Tiêu đề của các cột
    	    csvData.append("\"Customer ID\",\"Name\",\"Sex\",\"Birthday\",\"Address\",\"Email\"\n");
    	    
    	    // Thêm dữ liệu khách hàng vào CSV
    	    for (CustomerDto customer : customerList) {
    	        csvData.append("\"").append(customer.getCustomerID()).append("\",")
    	               .append("\"").append(customer.getCustomerName()).append("\",")
    	               .append("\"").append(customer.getSex() == "0" ? "Female" : "Male").append("\",")
    	               .append("\"").append(customer.getBirthday()).append("\",")
    	               .append("\"").append(customer.getAddress()).append("\",")
    	               .append("\"").append(customer.getEmail()).append("\"\n");
    	    }
    	    
    	    // Đường dẫn đến nơi lưu file CSV
    	    String filePath = "C:/Users/ASUS/Desktop/Data/customer_export.csv";
    	    
    	    try {
    	        // Ghi dữ liệu CSV vào file trên ổ đĩa
    	        java.nio.file.Files.write(java.nio.file.Paths.get(filePath), csvData.toString().getBytes());
    	        System.out.println("File exported successfully to " + filePath);
    	    } catch (Exception e) {
    	        e.printStackTrace();
    	    }
    	    
    	    // Chuyển hướng hoặc thông báo thành công (có thể thêm redirect hoặc thông báo trong giao diện)
    	    return mapping.findForward("search");
    	}
    	
    	if("SettingHeader".equals(action)) {
    		System.out.println("day la "+action);
    		return mapping.findForward("settingheader");
    	}

    	if ("Search".equals(action)) {
    	    // Kiểm tra định dạng YYYY/MM/DD của birthdayFrom và birthdayTo
    	    String datePattern = "^\\d{4}/\\d{2}/\\d{2}$";  // Regex kiểm tra định dạng YYYY/MM/DD}
    	    
    	    // Không có lỗi, thực hiện tìm kiếm
    	    customerList = searchDao.searchCustomers(customerName, sex, fromBirthday, toBirthday);
    	    
    	    // Logic phân trang...
    	    int PAGE_SIZE = 2;
    	    String pageStr = request.getParameter("page");
    	    Integer currentPage = 1;
    	    if (pageStr != null) {
    	        currentPage = Integer.parseInt(pageStr);
    	    }
    	    if (currentPage == null || currentPage < 1) {
    	        currentPage = 1;
    	    }
    	    int startRow = (currentPage - 1) * PAGE_SIZE;
    	    int totalCustomers = customerList.size();
    	    int totalPages = (int) Math.ceil((double) totalCustomers / PAGE_SIZE);
    	    
    	    List<CustomerDto> paginatedList = customerList.subList(
    	        Math.min(startRow, totalCustomers),
    	        Math.min(startRow + PAGE_SIZE, totalCustomers)
    	    );
    	    
    	    // Lưu thông tin tìm kiếm vào session
    	    session.setAttribute("customerName", customerName);
    	    session.setAttribute("sex", sex);
    	    session.setAttribute("fromBirthday", fromBirthday);
    	    session.setAttribute("toBirthday", toBirthday);
    	    
    	    session.setAttribute("customerList", paginatedList);
    	    session.setAttribute("currentPage", currentPage);
    	    session.setAttribute("totalPages", totalPages);
    	    
    	    return mapping.findForward("search");
    	}


        if (customerName != null || sex != null || fromBirthday != null || toBirthday != null) {
            customerList = searchDao.searchCustomers(customerName, sex, fromBirthday, toBirthday);
        } else {	
            customerList = searchDao.getAllCustomers();
        }
        int PAGE_SIZE = 2;
        String pageStr = request.getParameter("page");
        Integer currentPage = (Integer) session.getAttribute("currentPage");
        if (pageStr != null) {
            currentPage = Integer.parseInt(pageStr);
        }
        if (currentPage == null || currentPage < 1 ) {
            currentPage = 1;
        }
        int startRow = (currentPage - 1) * PAGE_SIZE;
        int totalCustomers = customerList.size();
        int totalPages = (int) Math.ceil((double) totalCustomers / PAGE_SIZE);
        List<CustomerDto> paginatedList = customerList.subList(
                Math.min(startRow, totalCustomers),
                Math.min(startRow + PAGE_SIZE, totalCustomers)
        );
        session.setAttribute("customerName", customerName);
        session.setAttribute("sex", sex);
        session.setAttribute("fromBirthday", fromBirthday);
        session.setAttribute("toBirthday", toBirthday);
        session.setAttribute("customerList", paginatedList);
        session.setAttribute("currentPage", currentPage);
        session.setAttribute("totalPages", totalPages);
    	return mapping.findForward("search");
    }
}

```
```
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c"%>
<%@ taglib uri="http://struts.apache.org/tags-html" prefix="html" %>
<%@ taglib uri="/WEB-INF/struts-bean.tld" prefix="bean" %>
<%@ taglib uri="/WEB-INF/struts-logic.tld" prefix="logic" %>
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>Search Customers</title>
<style type="text/css">
    body {
        margin-left: 20px;
        margin-right: 20px;
        background-color: #bcffff;
    }
    .header {
        border-bottom: 2px solid;
    }
    .header h1 {
        color: red;
    }
    .welcome_user {
        display: flex;
        align-items: center;
        justify-content: space-between;
    }
    .divider {
        height: 20px;
        width: 100%;
        background-color: #3097ff;
    }
    .search form {
        display: flex;
        justify-content: space-around;
        align-items: center;
        padding: 10px;
        margin-top: 20px;
        background-color: #ffff56;
    }
    .pagination form {
        margin-top: 10px;
        display: flex;
    }
    .pagination .navigation-left {
        display: flex;
        gap: 5px;
        align-items: center;
    }
    .pagination .navigation-right {
        display: flex;
        gap: 5px;
        margin-left: auto;
        align-items: center;
    }
    table {
        width: 100%;
        border-collapse: collapse;
    }
    th, td {
        padding: 8px;
        text-align: left;
        border-bottom: 1px solid #ddd;
    }
    th {
        background-color: #f2f2f2;
    }
    tr:first-child > th {
        background-color: rgb(0, 223, 0);
    }
    tr:nth-child(even) {
        background-color: #f2f2f2;
    }
</style>
</head>
<body>
    <div class="header">
        <h1>TRAINING</h1>
    </div>
    <div class="container">
        <div class="welcome_user">
            <p>Welcome, <%=session.getAttribute("userName")%></p>
            <a href="#">Log out</a>
        </div>
        <div class="divider"></div>
        <div class="search">
            <html:form action="/search" method="post">
                <div>
                    <label for="search">Customer Name:</label>
                    <html:text property="customerName" />
                </div>
                <div>
                    <label for="search">Sex:</label>
                    <html:select property="sex">
                        <html:option value=" "></html:option>
                        <html:option value="0">Male</html:option>
                        <html:option value="1">Female</html:option>
                    </html:select>
                </div>
                <div style="display: flex; justify-content: center; align-items: center;">
                    <label for="search">Birthday:</label>
                    <html:text property="birthdayFrom" styleClass="birthday" />
                    <p>~</p>
                    <html:text property="birthdayTo" styleClass="birthday" />
                </div>
                <button type="submit" value="Search" name="action">Search</button>
                <!-- Export Button -->
                <button type="submit" value="export" name="action">Export CSV</button>
                <button type="submit" value="SettingHeader" name="action">SettingHeader</button>
            </html:form>
        </div>
        <div class="pagination">
            <html:form action="/search" method="post">
                <html:hidden property="customerName" value="${customerName}" />
                <html:hidden property="sex" value="${sex}" />
                <html:hidden property="birthdayFrom" value="${fromBirthday}" />
                <html:hidden property="birthdayTo" value="${toBirthday}" />
                <div class="navigation-left">
                    <logic:present name="currentPage">
                        <logic:greaterThan name="currentPage" value="1">
                            <button type="submit" name="page" value="1">&laquo;</button>
                        </logic:greaterThan>
                        <logic:lessEqual name="currentPage" value="1">
                            <button type="button" disabled>&laquo;</button>
                        </logic:lessEqual>
                    </logic:present>

                    <logic:present name="currentPage">
                        <logic:greaterThan name="currentPage" value="1">
                            <button type="submit" name="page" value="${currentPage - 1}">&lt;</button>
                        </logic:greaterThan>
                        <logic:lessEqual name="currentPage" value="1">
                            <button type="button" disabled>&lt;</button>
                        </logic:lessEqual>
                    </logic:present>
                    <p>Previous</p>
                </div>
                <div class="navigation-right">
                    <p>Next</p>
                    <logic:present name="currentPage">
                        <logic:lessThan name="currentPage" value="${totalPages}">
                            <button type="submit" name="page" value="${currentPage + 1}">&gt;</button>
                        </logic:lessThan>
                        <logic:greaterEqual name="currentPage" value="${totalPages}">
                            <button type="button" disabled>&gt;</button>
                        </logic:greaterEqual>
                    </logic:present>

                    <logic:present name="currentPage">
                        <logic:lessThan name="currentPage" value="${totalPages}">
                            <button type="submit" name="page" value="${totalPages}">&raquo;</button>
                        </logic:lessThan>
                        <logic:greaterEqual name="currentPage" value="${totalPages}">
                            <button type="button" disabled>&raquo;</button>
                        </logic:greaterEqual>
                    </logic:present>
                </div>
            </html:form>
        </div>
        <div class="table">
            <html:form action="/search" method="post" onsubmit="return validateDelete();">
                <table>
				    <tr>
				        <th><input type="checkbox" id="select-all" onclick="selectAllCheckboxes(this)"></th>
				        <logic:iterate id="column" name="selectedColumns">
				            <th><bean:write name="column" /></th>
				        </logic:iterate>
				    </tr>
				    <logic:iterate id="customer" name="customerList">
				        <tr>
				            <td><input type="checkbox" name="deleteIds" value="${customer.customerID}" onclick="updateSelectAll()"></td>
				            <logic:iterate id="column" name="selectedColumns" scope="session">
				                <td>
				                    <logic:equal name="column" value="Customer ID">
				                        <a href="edit?id=${customer.customerID}"><bean:write name="customer" property="customerID" /></a>
				                    </logic:equal>
				                    <logic:equal name="column" value="Customer Name">
				                        <bean:write name="customer" property="customerName" />
				                    </logic:equal>
				                    <logic:equal name="column" value="Sex">
				                        <logic:equal name="customer" property="sex" value="0">Male</logic:equal>
				                        <logic:equal name="customer" property="sex" value="1">Female</logic:equal>
				                    </logic:equal>
				                    <logic:equal name="column" value="Birthday">
				                        <bean:write name="customer" property="birthday" />
				                    </logic:equal>
				                    <logic:equal name="column" value="Address">
				                        <bean:write name="customer" property="address" />
				                    </logic:equal>
				                    <logic:equal name="column" value="Email">
				                        <bean:write name="customer" property="email" />
				                    </logic:equal>
				                </td>
				            </logic:iterate>
				        </tr>
				    </logic:iterate>
                </table>
                <div style="padding-block: 20px; display: flex; gap: 20px">
                    <button id="addNew" type="button">Add New</button>
                    <button id="deleteButton" type="submit" name="action" value="delete">Delete</button>
                </div>
            </html:form>
        </div>
	
    </div>
</body>
</html>

```
```
upload.jsp
<%@ taglib uri="http://struts.apache.org/tags-html" prefix="html" %>
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c" %>
<%@ taglib uri="/WEB-INF/struts-bean.tld" prefix="bean" %>
<%@ taglib uri="/WEB-INF/struts-logic.tld" prefix="logic" %>
<html>
<head>
    <title>Setting Header</title>
    <style>
        .selected {
            background-color: lightblue; /* Màu nền khi được chọn */
        }
        label {
            display: block; /* Hiển thị mỗi label trên 1 dòng */
            padding: 5px;
            cursor: pointer;
        }
        .list-container {
            display: flex;
            justify-content: space-between;
        }
        .list {
            width: 200px;
            height: 200px;
            border: 1px solid black;
            padding: 10px;
            overflow-y: auto;
        }
    </style>
    <script>
	    function checkButtonState() {
	        var customerList = document.getElementById('customerList');
	
	            // Nếu label được chọn là đầu tiên, disable Move Up
	            document.getElementById('moveUpBtn').disabled = (selectedLabel.previousElementSibling === null);
	
	            // Nếu label được chọn là cuối cùng, disable Move Down
	            document.getElementById('moveDownBtn').disabled = (selectedLabel.nextElementSibling === null);
	
	            // Các nút Move Left và Move Right chỉ cần kiểm tra nếu có label được chọn
	            document.getElementById('moveLeftBtn').disabled = false;
	            document.getElementById('moveRightBtn').disabled = false;
	    }
        function toggleMoveButtons() {
            var selectedLabel = document.querySelector('#customerList label.selected');
            var moveUpButton = document.getElementById('moveUpButton');
            var moveDownButton = document.getElementById('moveDownButton');

            if (!selectedLabel) {
                return;
            }

            // Disable 'Move Up' if at the top
            if (!selectedLabel.previousElementSibling) {
                moveUpButton.disabled = true;
            } else {
                moveUpButton.disabled = false;
            }

            // Disable 'Move Down' if at the bottom
            if (!selectedLabel.nextElementSibling) {
                moveDownButton.disabled = true;
            } else {
                moveDownButton.disabled = false;
            }
        }
	    function selectLabel(label) {
	        // Bỏ chọn tất cả các label trong cả hai danh sách
	        var allLabels = document.querySelectorAll('.list label');
	        allLabels.forEach(function(item) {
	            item.classList.remove('selected');
	        });
	        // Chọn label hiện tại
	        label.classList.add('selected');
	    }

        function moveUp() {
            var selectedLabel = document.querySelector('#customerList label.selected');
            if (selectedLabel && selectedLabel.previousElementSibling) {
                selectedLabel.parentNode.insertBefore(selectedLabel, selectedLabel.previousElementSibling);
                toggleMoveButtons();
            }
        }

        function moveDown() {
            var selectedLabel = document.querySelector('#customerList label.selected');
            if (selectedLabel && selectedLabel.nextElementSibling) {
                selectedLabel.parentNode.insertBefore(selectedLabel.nextElementSibling, selectedLabel);
            }
        }

        function moveRight() {
            var selectedLabel = document.querySelector('#columnList label.selected');
            if (selectedLabel) {
                document.getElementById('customerList').appendChild(selectedLabel);
            }
        }

        function moveLeft() {
            var selectedLabel = document.querySelector('#customerList label.selected');
            if (selectedLabel) {
                document.getElementById('columnList').appendChild(selectedLabel);
            } else {
                alert('Please select a label to move left.');
            }
        }
        function saveSelectedColumns() {
            var selectedColumns = [];
            var customerList = document.getElementById('customerList');
            var labels = customerList.querySelectorAll('label');
            
            labels.forEach(function(label) {
                selectedColumns.push(label.textContent.trim());
            });
            
            document.getElementById('selectedColumns').value = selectedColumns.join(',');
        }
    </script>
</head>
<body>
    <h2>Manage Search Screen Columns</h2>

    <html:form action="/upload">
        <div class="list-container">
            <!-- Column List -->
            <div id="columnList" class="list">
                <logic:iterate id="column" name="availableColumns">
                    <label onclick="selectLabel(this)">
                        <bean:write name="column" />
                    </label>
                </logic:iterate>
            </div>

            <!-- Move Buttons -->
            <div>
                <button type="button" onclick="moveUp()">Move Up</button><br><br>
                <button type="button" onclick="moveDown()">Move Down</button><br><br>
                <button type="button" onclick="moveRight()">Move Right</button><br><br>
                <button type="button" onclick="moveLeft()">Move Left</button>
            </div>

            <!-- Customer List -->
            <div id="customerList" class="list">
                <c:if test="${not empty sessionScope.selectedColumns}">
                    <c:forEach var="selectedColumn" items="${sessionScope.selectedColumns}">
                        <label onclick="selectLabel(this)">
                            <bean:write name="selectedColumn" />
                        </label>
                    </c:forEach>
                </c:if>
            </div>
        </div>

        <!-- Hidden input to store the selected columns -->
        <input type="hidden" name="selectedColumns" id="selectedColumns" 
               value="<%= request.getSession().getAttribute("selectedColumns") != null ? 
                       String.join(",", (String[]) request.getSession().getAttribute("selectedColumns")) : "conchonay" %>" />

        <br><br>
        <button type="submit" onclick="saveSelectedColumns()" name=action value="save">save</button>
    </html:form>
</body>
</html>

```
```
setting action

package fjs.cs.action;

import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import javax.servlet.http.HttpSession;

import org.apache.struts.action.Action;
import org.apache.struts.action.ActionForm;
import org.apache.struts.action.ActionForward;
import org.apache.struts.action.ActionMapping;

import fjs.cs.dao.SearchDao;

public class UploadAction extends Action {
    private SearchDao searchDao;

    public void setSearchDao(SearchDao searchDao) {
        this.searchDao = searchDao;
    }

    public ActionForward execute(ActionMapping mapping, ActionForm form,
                                 HttpServletRequest request, HttpServletResponse response) {
        // Danh sách tất cả các cột
    	HttpSession session = request.getSession();
        String[] allColumns = {"Customer ID", "Customer Name", "Sex", "Birthday", "Address", "Email"};
        String selectedColumns = request.getParameter("selectedColumns");
        String action = request.getParameter("action");
        String[] avaiDefault = (String[]) session.getAttribute("availableColumns");
        if(avaiDefault==null) {
            List<String> defaultColumns = Arrays.asList("Customer ID", "Customer Name", "Sex", "Birthday", "Address");

            // Các cột còn lại (bao gồm "Email") sẽ là cột chưa được chọn
            List<String> availableList = new ArrayList<>();
            for (String column : allColumns) {
                if (!defaultColumns.contains(column)) {
                    availableList.add(column);
                }
            }

            // Lưu danh sách cột chưa được chọn vào session
            session.setAttribute("availableColumns", availableList.toArray(new String[0]));
        }
        List<String> selectedList = new ArrayList<>();
        List<String> availableList = new ArrayList<>();
        if("save".equals(action)) {
            // Chuyển danh sách cột đã chọn thành danh sách
            if (selectedColumns != null && !selectedColumns.isEmpty()) {
                selectedList = Arrays.asList(selectedColumns.split(","));
                // Lưu danh sách cột đã chọn vào session
                request.getSession().setAttribute("selectedColumns", selectedList.toArray(new String[0]));
            }

            // Các cột chưa được chọn (cột ẩn) = Tất cả các cột - Cột đã chọn
            for (String column : allColumns) {
                if (!selectedList.contains(column)) {
                    availableList.add(column);
                    System.out.println(column);
                }
            }

            // Lưu danh sách cột ẩn vào session
            request.getSession().setAttribute("availableColumns", availableList.toArray(new String[0]));
            // Chuyển đến trang JSP setting header
            return mapping.findForward("success");
        }
//        else {
//            // Nếu action không phải là "save", thiết lập availableColumns là mảng rỗng
//            session.setAttribute("availableColumns", new String[0]);
//        }
        return mapping.findForward("success");
    }
}

```
------ new------
```
chua test
<%@ taglib uri="http://struts.apache.org/tags-html" prefix="html" %>
<%@ taglib uri="http://struts.apache.org/tags-bean" prefix="bean" %>
<%@ taglib uri="http://struts.apache.org/tags-logic" prefix="logic" %>

<html:form action="/updateColumnSettings">
  <div class="settings-container">
    <!-- Label cho các cột đang hiển thị -->
    <div class="column-group">
      <h4>Columns Showing:</h4>
      <html:select property="selectedColumns" multiple="true" size="5" styleClass="column-select">
        <html:options collection="availableColumns" property="value" labelProperty="label"/>
      </html:select>
    </div>

    <!-- Nút chuyển -->
    <div class="transfer-buttons">
      <button type="button" onclick="moveColumns('right')">&gt;&gt;</button>
      <button type="button" onclick="moveColumns('left')">&lt;&lt;</button>
    </div>

    <!-- Label cho các cột bị ẩn -->
    <div class="column-group">
      <h4>Hidden Columns:</h4>
      <html:select property="hiddenColumns" multiple="true" size="5" styleClass="column-select">
        <html:options collection="hiddenColumns" property="value" labelProperty="label"/>
      </html:select>
    </div>
  </div>

  <!-- Table hiển thị dữ liệu -->
  <table id="dataTable">
    <thead>
      <tr>
        <logic:iterate id="column" name="selectedColumns">
          <th class="column-${column.value}"><bean:write name="column" property="label"/></th>
        </logic:iterate>
      </tr>
    </thead>
    <tbody>
      <logic:iterate id="row" name="dataList">
        <tr>
          <logic:iterate id="column" name="selectedColumns">
            <td class="column-${column.value}">
              <bean:write name="row" property="${column.value}"/>
            </td>
          </logic:iterate>
        </tr>
      </logic:iterate>
    </tbody>
  </table>
</html:form>
```
```
.settings-container {
  display: flex;
  align-items: center;
  margin-bottom: 20px;
}

.column-group {
  flex: 1;
}

.column-select {
  width: 200px;
  height: 150px;
}

.transfer-buttons {
  display: flex;
  flex-direction: column;
  padding: 0 20px;
}

.transfer-buttons button {
  margin: 5px;
  padding: 5px 10px;
}

table {
  width: 100%;
  border-collapse: collapse;
}

th, td {
  border: 1px solid #ddd;
  padding: 8px;
  text-align: left;
}
```
```
function moveColumns(direction) {
  var sourceSelect = direction === 'right' ? 
    document.getElementsByName('selectedColumns')[0] : 
    document.getElementsByName('hiddenColumns')[0];
  
  var targetSelect = direction === 'right' ? 
    document.getElementsByName('hiddenColumns')[0] : 
    document.getElementsByName('selectedColumns')[0];

  // Di chuyển các options đã chọn
  for(var i = sourceSelect.options.length - 1; i >= 0; i--) {
    var option = sourceSelect.options[i];
    if(option.selected) {
      targetSelect.add(option);
      updateTableColumns();
    }
  }
}

function updateTableColumns() {
  // Lấy danh sách các cột đang được chọn
  var selectedColumns = document.getElementsByName('selectedColumns')[0].options;
  var columnNames = Array.from(selectedColumns).map(opt => opt.value);
  
  // Ẩn/hiện các cột tương ứng
  var table = document.getElementById('dataTable');
  var headers = table.getElementsByTagName('th');
  var cells = table.getElementsByTagName('td');
  
  // Cập nhật hiển thị của các cột
  Array.from(headers).concat(Array.from(cells)).forEach(cell => {
    var columnClass = cell.className;
    var columnName = columnClass.replace('column-', '');
    cell.style.display = columnNames.includes(columnName) ? '' : 'none';
  });
}

// Tự động lưu settings khi có thay đổi
function saveSettings() {
  var selectedColumns = Array.from(document.getElementsByName('selectedColumns')[0].options)
    .map(opt => opt.value);
  
  // Gọi AJAX để lưu settings
  var xhr = new XMLHttpRequest();
  xhr.open('POST', 'saveColumnSettings.do', true);
  xhr.setRequestHeader('Content-Type', 'application/json');
  xhr.send(JSON.stringify({ columns: selectedColumns }));
}
```
```
public class ColumnSettingsAction extends Action {
    
    public ActionForward execute(ActionMapping mapping, ActionForm form,
            HttpServletRequest request, HttpServletResponse response) throws Exception {
        
        // Khởi tạo danh sách các cột có sẵn
        List<LabelValueBean> availableColumns = new ArrayList<>();
        availableColumns.add(new LabelValueBean("ID", "id"));
        availableColumns.add(new LabelValueBean("Name", "name"));
        availableColumns.add(new LabelValueBean("Sex", "sex"));
        availableColumns.add(new LabelValueBean("Email", "email"));
        availableColumns.add(new LabelValueBean("Address", "address"));
        
        // Lấy settings từ database hoặc session
        UserSettings settings = getUserSettings(request);
        
        // Phân chia cột hiện/ẩn dựa trên settings
        List<LabelValueBean> selectedColumns = new ArrayList<>();
        List<LabelValueBean> hiddenColumns = new ArrayList<>();
        
        for(LabelValueBean column : availableColumns) {
            if(settings.isColumnVisible(column.getValue())) {
                selectedColumns.add(column);
            } else {
                hiddenColumns.add(column);
            }
        }
        
        // Set attributes
        request.setAttribute("availableColumns", selectedColumns);
        request.setAttribute("hiddenColumns", hiddenColumns);
        
        return mapping.findForward("success");
    }
}
```
```
public class ColumnSettingsForm extends ActionForm {
    private String[] selectedColumns;
    private String[] hiddenColumns;
    
    // Getters and setters
    public String[] getSelectedColumns() {
        return selectedColumns;
    }
    
    public void setSelectedColumns(String[] selectedColumns) {
        this.selectedColumns = selectedColumns;
    }
    
    public String[] getHiddenColumns() {
        return hiddenColumns;
    }
    
    public void setHiddenColumns(String[] hiddenColumns) {
        this.hiddenColumns = hiddenColumns;
    }
}
```
neeeeee
```
upload action
package fjs.cs.action;

import java.io.BufferedReader;
import java.io.InputStream;
import java.io.InputStreamReader;
import java.util.ArrayList;
import java.util.List;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import org.apache.struts.action.Action;
import org.apache.struts.action.ActionForm;
import org.apache.struts.action.ActionForward;
import org.apache.struts.action.ActionMapping;
import org.apache.struts.action.ActionMessage;
import org.apache.struts.action.ActionMessages;
import org.apache.struts.upload.FormFile;

import fjs.cs.form.UploadForm;
import fjs.cs.dao.SearchDao;
import fjs.cs.dto.CustomerDto;

public class UploadAction extends Action {
    private SearchDao searchDao;
	
    public void setSearchDao(SearchDao searchDao) {
        this.searchDao = searchDao;
    }
    public ActionForward execute(ActionMapping mapping, ActionForm form,
                                 HttpServletRequest request, HttpServletResponse response) {
        UploadForm uploadForm = (UploadForm) form;
        FormFile file = uploadForm.getFile();
        String action = request.getParameter("action");

		// Kiểm tra khi ấn nút "Upload"
		if ("Upload".equals(action)) {
		// Kiểm tra nếu không có file được chọn
		if (file == null || file.getFileSize() == 0) {
		ActionMessages errors = new ActionMessages();
		errors.add("file", new ActionMessage("error.file.notexist"));
		saveErrors(request, errors);
		return mapping.findForward("success");  // Quay lại trang hiện tại khi có lỗi
		}
		
		// Đặt fileName vào form nếu file hợp lệ
		String fileName = file.getFileName();
		uploadForm.setFileName(fileName);
		ActionMessages errors = new ActionMessages();
		List<CustomerDto> customers = readCustomersFromFile(file, errors);
		
		// Kiểm tra nếu có lỗi
		if (!errors.isEmpty()) {
		saveErrors(request, errors);
		return mapping.findForward("success");  // Quay lại trang hiện tại khi có lỗi
		}
		
		// Biến lưu index của các dòng được insert và update
		List<Integer> insertedLines = new ArrayList<>();
		List<Integer> updatedLines = new ArrayList<>();
		
		// Thêm hoặc cập nhật từng customer trong danh sách
		int lineNumber = 1;
		for (CustomerDto customer : customers) {
		if ("male".equalsIgnoreCase(customer.getSex())) {
		   customer.setSex("0"); // Male thành 0
		} else if ("female".equalsIgnoreCase(customer.getSex())) {
		   customer.setSex("1"); // Female thành 1
		}
		
		// Kiểm tra nếu có customerId thì update, nếu không có thì add mới
		if (customer.getCustomerID() != 0) {
		   // Kiểm tra xem customer đã tồn tại chưa
		   if (isCustomerExists(customer.getCustomerID())) {
		       searchDao.editCustomer(customer);  // Gọi hàm editCustomer nếu đã tồn tại
		       updatedLines.add(lineNumber); // Thêm index dòng được update vào danh sách
		   } else {
		       searchDao.addCustomer(customer);   // Gọi hàm addCustomer nếu không tồn tại
		       insertedLines.add(lineNumber); // Thêm index dòng được insert vào danh sách
		   }
		} else {
		   // Nếu không có customerId thì thêm mới
		   searchDao.addCustomer(customer);
		   insertedLines.add(lineNumber); // Thêm index dòng được insert vào danh sách
		}
		lineNumber++;
		}
		
		// Tạo thông báo thành công
		ActionMessages successMessages = new ActionMessages();
		successMessages.add("success", new ActionMessage("message.success.general"));  // Thêm thông báo "Successfully"
		
		if (!insertedLines.isEmpty()) {
			System.out.println("no vo");
			successMessages.add("insertSuccess", 
		    new ActionMessage("message.success.insert", insertedLines.toString().replaceAll("[\\[\\]]", "")));
		}
		if (!updatedLines.isEmpty()) {
			System.out.println("no cung vo");
			successMessages.add("updateSuccess", 
		    new ActionMessage("message.success.update", updatedLines.toString().replaceAll("[\\[\\]]", "")));
		}
		
		// Lưu thông báo thành công vào request
		saveMessages(request, successMessages);
		
		// Tiến hành xử lý tiếp nếu file hợp lệ
		return mapping.findForward("success");
		}

        // Nếu không ấn nút "Upload", trả về trang hiện tại
        return mapping.findForward("success");
    }
    private List<CustomerDto> readCustomersFromFile(FormFile file, ActionMessages errors) {
        List<CustomerDto> customers = new ArrayList<>();
        InputStream inputStream = null;
        BufferedReader reader = null;

        try {
            inputStream = file.getInputStream();
            reader = new BufferedReader(new InputStreamReader(inputStream));

            String line;
            // Bỏ qua dòng tiêu đề
            reader.readLine();

            int lineNumber = 1; // Đếm số dòng để xác định dòng lỗi
            while ((line = reader.readLine()) != null) {
                String[] values = line.split(","); // Chia tách theo dấu phẩy

                // Kiểm tra độ dài mảng để đảm bảo không có giá trị thiếu
                if (values.length >= 6) {
                    String customerIdStr = values[0].trim();
                    String customerName = values[1].trim();
                    String sex = values[2].trim();
                    String birthday = values[3].trim();
                    String email = values[4].trim();
                    String address = values[5].trim();

                    CustomerDto customer = null;
                    boolean hasErrors = false; // Biến để theo dõi có lỗi hay không

                    // Kiểm tra nếu customerName là rỗng
                    if (customerName.isEmpty()) {
                        errors.add("customerName", new ActionMessage("error.customer.name.empty", lineNumber));
                        hasErrors = true; // Đánh dấu có lỗi
                    }
                    // Kiểm tra nếu email lớn hơn 40 ký tự
                    if (email.length() > 40) {
                        errors.add("email", new ActionMessage("error.email.tooLong", lineNumber));
                        hasErrors = true; // Đánh dấu có lỗi
                    }

                    // Nếu không có lỗi, kiểm tra customerId
                    if (!hasErrors) {
                        if (!customerIdStr.isEmpty()) {
                            int customerId = Integer.parseInt(customerIdStr);
                            // Kiểm tra sự tồn tại của customerId trong cơ sở dữ liệu
//                            if (!isCustomerExists(customerId)) {
//                                // Nếu deleteYmd khác null, thêm thông báo lỗi
//                                errors.add("customerIdNotExists", new ActionMessage("error.customer.not.exists", lineNumber, customerId));
//                                hasErrors = true; // Đánh dấu có lỗi
//                            } else {
                                customer = new CustomerDto(customerId, customerName, sex, birthday, email, address);
//                            }
                        } else {
                            // Tạo CustomerDto không bao gồm customerId
                            customer = new CustomerDto(customerName, sex, birthday, email, address);
                        }
                    }

                    // Nếu không có lỗi, thêm customer vào danh sách
                    if (!hasErrors && customer != null) {
                        customers.add(customer);
                    }
                }
                lineNumber++;
            }
        } catch (Exception e) {
            e.printStackTrace(); // Ghi log hoặc xử lý lỗi khác
        } finally {
            try {
                if (reader != null) {
                    reader.close();
                }
                if (inputStream != null) {
                    inputStream.close();
                }
            } catch (Exception e) {
                e.printStackTrace(); // Xử lý lỗi đóng file
            }
        }

        return customers;
    }

    private boolean isCustomerExists(int customerId) {
        // Phương thức kiểm tra xem customerId có tồn tại trong cơ sở dữ liệu hay không
        // Giả sử bạn đã có một lớp DAO để truy vấn dữ liệu
        return searchDao.isCustomerExists(customerId);
    }
}


```
```
message.success.general=Successfully \\n
message.success.insert=Insert line(s): {0} \\n
message.success.update=Update line(s): {0} \\n
```
```
<%@ taglib uri="http://struts.apache.org/tags-html" prefix="html" %>
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c" %>
<%@ taglib uri="http://struts.apache.org/tags-html" prefix="html" %>
<%@ taglib uri="/WEB-INF/struts-bean.tld" prefix="bean" %>
<%@ taglib uri="/WEB-INF/struts-logic.tld" prefix="logic" %>
<html>
<head>
    <title>Upload File</title>
    <style>
        .container {
            display: flex;
            justify-content: space-between;
            width: 400px;
        }
        .hidden-file-input {
            display: none;
        }
        .custom-button {
            background-color: #4CAF50;
            color: white;
            border: none;
            padding: 10px;
            cursor: pointer;
        }
    </style>
    <script>
        function triggerFileInput() {
            document.getElementById('fileInput').click();
        }
        function updateFileName() {
            var input = document.getElementById('fileInput');
            var fileName = input.files[0] ? input.files[0].name : '';
            document.getElementById('fileName').value = fileName;
        }
        window.onload = function() {
            setTimeout(function() {
                var errorMessages = document.getElementById("htmlErrors").innerText.trim();
                if (errorMessages) {
                	alert(errorMessages.replace(/\\n/g, '\n'));
                }
            }, 100); // Thời gian chờ 2000ms (2 giây)
            // Hiển thị alert cho thành công
            setTimeout(function() {
                var successMessages = document.getElementById("messages").innerText.trim();
                if (successMessages) {
                	alert(successMessages.replace(/\\n/g, '\n'));
                }
            }, 100); // Thời gian chờ 2000ms (2 giây)
        }
    </script>
</head>
<body>
    <h2>Upload File Example</h2>
    <p id="messageParagraph">Hahahahea</p>
    <p id="testP"></p>
    <html:errors/>
    <h2>Example</h2>
    <!-- Thẻ chứa html:errors nhưng ẩn đi -->
    <div id="htmlErrors" style="display:none;">
        <html:errors />
    </div>
    <!-- Thẻ chứa thông báo thành công nhưng ẩn đi -->
    <c:out value="${successMessages}" />
    <div id= messages>
        <html:messages id="aMsg" message="true">
    		<bean:write name="aMsg" filter="false" />
    	</html:messages>
    </div>
    <html:form action="/upload" enctype="multipart/form-data" onsubmit="showParagraphContent();">
        <div class="container">
            <input type="text" id="fileName" name="fileName" value="<%= request.getAttribute("fileName") != null ? request.getAttribute("fileName").toString() : "" %>" readonly="true" />
            <input type="file" id="fileInput" class="hidden-file-input" name="file" onchange="updateFileName()" />
            <button type="button" class="custom-button" onclick="triggerFileInput()">Browse</button>
        </div>
        <br/>
        <button type="submit" value="Upload" name="action">Upload</button>
    </html:form>
</body>
</html>

```
https://drive.google.com/drive/folders/1bPh22WOHnfjV8E2gzLIvzQr1Bx1uL9nx?usp=sharing
```

uploadform
package fjs.cs.form;

import org.apache.struts.action.ActionForm;
import org.apache.struts.upload.FormFile;

public class UploadForm extends ActionForm {
    /**
	 * 
	 */
	private static final long serialVersionUID = 1L;
	private FormFile file;
    private String fileName;

    public FormFile getFile() {
        return file;
    }

    public void setFile(FormFile file) {
        this.file = file;
    }

    public String getFileName() {
        return fileName;
    }

    public void setFileName(String fileName) {
        this.fileName = fileName;
    }
}

```
```
error.username.required=ChÆ°a nháº­p user name
error.password.required=ChÆ°a nháº­p password
error.login.invalid= ããã«ã¡ã¯
error.birthday.invalid=ngay sinh khong hop le
error.delete.noselection=Please select at least one customer to delete.
error.file.notexist=File import not existed
error.customer.name.empty = Line {0} : Customer name is empty \\n
error.email.tooLong = Line {0} : value email is more than 40
error.customer.not.exists=Line {0}: customerId = {1} is not existed
dao
package fjs.cs.dao;

import java.util.ArrayList;
import java.util.List;

import org.hibernate.Criteria;
import org.hibernate.Query;
import org.hibernate.Session;
import org.hibernate.SessionFactory;
import org.hibernate.Transaction;

import fjs.cs.dto.CustomerDto;
import fjs.cs.model.Customer;

public class SearchDao {
	
	private SessionFactory sessionFactory;

	public void setSessionFactory(SessionFactory sessionFactory) {
	   this.sessionFactory = sessionFactory;
	}
	@SuppressWarnings("unchecked")
	public List<CustomerDto> getAllCustomers() {
	    Session session = sessionFactory.openSession();
	    String hql = "SELECT c.customerID, c.customerName, c.sex, c.birthday, c.address FROM Customer c WHERE c.delete_ymd IS NULL";
	    List<Object[]> results = session.createQuery(hql).list();
	    session.close();
	
	    List<CustomerDto> customers = new ArrayList<>();
	    for (Object[] row : results) {
	        CustomerDto dto = new CustomerDto();
	        dto.setCustomerID((int) row[0]);
	        dto.setCustomerName((String) row[1]);
	        dto.setSex((String) row[2]);
	        dto.setBirthday((String) row[3]);
	        dto.setAddress((String) row[4]);
	        customers.add(dto);
	    }
	
	    return customers;
	}		
//	@SuppressWarnings("unchecked")
//	public List<CustomerDto> getAllCustomers() {
//    	Transaction transaction = null;
//    	List<CustomerDto> customer = new ArrayList<>();
//        Session session = sessionFactory.openSession();
//        transaction = session.beginTransaction();
//        String hql = "SELECT new fjs.cs.dto.CustomerDto(c.customerID, c.customerName, c.sex, c.birthday, c.address) " +
//                "FROM Customer c WHERE c.delete_ymd IS NULL";
//        Query query = session.createQuery(hql);
//        customer  = query.list();
//        transaction.commit();
//        return customer;
//    }
	
    public List<Object[]> getAllCustomerNamesAndAddresses() {
        Transaction transaction = null;
        List<Object[]> customerList = null;
        Session session = sessionFactory.openSession();

        try {
            // Bắt đầu transaction
            transaction = session.beginTransaction();

            // Viết câu HQL để lấy customerName và address
            String hql = "SELECT c.customerName, c.address FROM Customer c WHERE c.delete_ymd IS NULL";
            Query query = session.createQuery(hql);  // Sử dụng kiểu Query không generic

            // Lấy danh sách kết quả và đảm bảo nó là List<Object[]>
            customerList = (List<Object[]>) query.list(); // Cast kết quả về List<Object[]>
            
            // Commit transaction
            transaction.commit();
        } catch (Exception e) {
            if (transaction != null) {
                transaction.rollback();
            }
            e.printStackTrace();
        } finally {
            session.close();
        }
        
        return customerList;
    }
	@SuppressWarnings("unchecked")
	public List<CustomerDto> searchCustomers(String customerName, String sex, String birthdayFrom, String birthdayTo) {

	    Session session = sessionFactory.openSession();
	    
	    try {
	        // Tạo câu truy vấn HQL
	        StringBuilder hql = new StringBuilder("SELECT c.customerID, c.customerName, c.sex, c.birthday, c.address FROM Customer c WHERE c.delete_ymd IS NULL");
	        List<Object> params = new ArrayList<>();

	        // Thêm điều kiện tìm kiếm
	        if (customerName != null && !customerName.trim().isEmpty()) {
	            hql.append(" AND c.customerName LIKE ?");
	            params.add("%" + customerName + "%");
	        }
	        
	        if (sex != null && !sex.trim().isEmpty()) {
	            hql.append(" AND c.sex = ?");
	            params.add(sex);
	        }
	        
	        if (birthdayFrom != null && !birthdayFrom.trim().isEmpty()) {
	            hql.append(" AND c.birthday >= ?");
	            params.add(birthdayFrom);
	        }
	        
	        if (birthdayTo != null && !birthdayTo.trim().isEmpty()) {
	            hql.append(" AND c.birthday <= ?");
	            params.add(birthdayTo);
	        }

	        // Thực thi query
	        Query query = session.createQuery(hql.toString());
	        for (int i = 0; i < params.size(); i++) {
	            query.setParameter(i, params.get(i));
	        }

	        // Lấy danh sách kết quả trả về
	        List<Object[]> results = query.list();  // Kết quả là danh sách Object[]
	        List<CustomerDto> customerDtos = new ArrayList<>();

	        // Xử lý kết quả trả về
	        for (Object[] row : results) {
	            CustomerDto dto = new CustomerDto(
	                (Integer) row[0],   // customerID
	                (String) row[1],    // customerName
	                (String) row[2],    // sex
	                (String) row[3],    // birthday
	                (String) row[4]     // address
	            );
	            customerDtos.add(dto);
	        }
	        
	        return customerDtos;
	    } finally {
	        session.close();
	    }
	}	
	public void addCustomer(CustomerDto customerDto) {
	    Transaction transaction = null;
	    Session session = sessionFactory.openSession();
	    
	    try {
	        // Bắt đầu transaction
	        transaction = session.beginTransaction();

	        // Tạo đối tượng Customer từ CustomerDto
	        Customer customer = new Customer();
	        customer.setCustomerID(customerDto.getCustomerID()); // Nếu ID tự động sinh, bạn có thể bỏ qua dòng này
	        customer.setCustomerName(customerDto.getCustomerName());
	        customer.setSex(customerDto.getSex());
	        customer.setBirthday(customerDto.getBirthday());
	        customer.setAddress(customerDto.getAddress());
	        customer.setEmail(customerDto.getEmail()); // Nếu có email trong Dto
	        
	        // Lưu đối tượng Customer vào database
	        session.save(customer);
	        
	        // Commit transaction
	        transaction.commit();
	        
	    } catch (Exception e) {
	        if (transaction != null) {
	            transaction.rollback(); // Rollback nếu có lỗi
	        }
	        e.printStackTrace();
	    } finally {
	        session.close(); // Đóng session sau khi hoàn thành
	    }
	}
    public boolean isCustomerExists(int customerId) {
    	Session session = sessionFactory.openSession();
        boolean exists = false;

        try {
            // HQL query để kiểm tra sự tồn tại của customerId
            String hql = "SELECT count(c.customerID) FROM Customer c WHERE c.customerID = :customerID AND c.delete_ymd IS NULL";
            Query query = session.createQuery(hql); // Không sử dụng kiểu tham số hóa ở đây
            query.setParameter("customerId", customerId);

            Long count = (Long) query.uniqueResult(); // Lấy kết quả duy nhất
            exists = (count != null && count > 0); // Nếu có kết quả và > 0, thì customerId tồn tại
        } catch (Exception e) {
            e.printStackTrace(); // Xử lý lỗi
        } finally {
            session.close(); // Đóng session
        }
        return exists;
    }
}

```
```
uploadaction
package fjs.cs.action;

import java.io.BufferedReader;
import java.io.InputStream;
import java.io.InputStreamReader;
import java.util.ArrayList;
import java.util.List;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import org.apache.struts.action.Action;
import org.apache.struts.action.ActionForm;
import org.apache.struts.action.ActionForward;
import org.apache.struts.action.ActionMapping;
import org.apache.struts.action.ActionMessage;
import org.apache.struts.action.ActionMessages;
import org.apache.struts.upload.FormFile;

import fjs.cs.form.UploadForm;
import fjs.cs.dao.SearchDao;
import fjs.cs.dto.CustomerDto

public class UploadAction extends Action {
    private SearchDao searchDao;
	
    public void setSearchDao(SearchDao searchDao) {
        this.searchDao = searchDao;
    }
    public ActionForward execute(ActionMapping mapping, ActionForm form,
                                 HttpServletRequest request, HttpServletResponse response) {
        UploadForm uploadForm = (UploadForm) form;
        FormFile file = uploadForm.getFile();
        String action = request.getParameter("action");

        // Kiểm tra khi ấn nút "Upload"
        if ("Upload".equals(action)) {
            // Kiểm tra nếu không có file được chọn
            if (file == null || file.getFileSize() == 0) {
                ActionMessages errors = new ActionMessages();
                errors.add("file", new ActionMessage("error.file.notexist"));
                saveErrors(request, errors);
                return mapping.findForward("success");  // Quay lại trang hiện tại khi có lỗi
            }

            // Đặt fileName vào form nếu file hợp lệ
            String fileName = file.getFileName();
            uploadForm.setFileName(fileName);
            ActionMessages errors = new ActionMessages();
            List<CustomerDto> customers = readCustomersFromFile(file,errors);
            // Kiểm tra nếu có lỗi
            if (!errors.isEmpty()) {
                saveErrors(request, errors);
                return mapping.findForward("success");  // Quay lại trang hiện tại khi có lỗi
            }
         // Thêm từng customer trong danh sách vào cơ sở dữ liệu
            for (CustomerDto customer : customers) {
                if ("male".equalsIgnoreCase(customer.getSex())) {
                    customer.setSex("0"); // Male thành 0
                } else if ("female".equalsIgnoreCase(customer.getSex())) {
                    customer.setSex("1"); // Female thành 1
                }
            	searchDao.addCustomer(customer);  // Gọi hàm addCustomer cho từng customer
            }
            // Tiến hành xử lý tiếp nếu file hợp lệ
            return mapping.findForward("search");
        }

        // Nếu không ấn nút "Upload", trả về trang hiện tại
        return mapping.findForward("success");
    }
    private List<CustomerDto> readCustomersFromFile(FormFile file, ActionMessages errors) {
        List<CustomerDto> customers = new ArrayList<>();
        InputStream inputStream = null;
        BufferedReader reader = null;

        try {
            inputStream = file.getInputStream();
            reader = new BufferedReader(new InputStreamReader(inputStream));

            String line;
            // Bỏ qua dòng tiêu đề
            reader.readLine();

            int lineNumber = 1; // Đếm số dòng để xác định dòng lỗi
            while ((line = reader.readLine()) != null) {
                String[] values = line.split(","); // Chia tách theo dấu phẩy

                // Kiểm tra độ dài mảng để đảm bảo không có giá trị thiếu
                if (values.length >= 6) {
                    String customerIdStr = values[0].trim();
                    String customerName = values[1].trim();
                    String sex = values[2].trim();
                    String birthday = values[3].trim();
                    String email = values[4].trim();
                    String address = values[5].trim();

                    CustomerDto customer = null;
                    boolean hasErrors = false; // Biến để theo dõi có lỗi hay không

                    // Kiểm tra nếu customerName là rỗng
                    if (customerName.isEmpty()) {
                        errors.add("customerName", new ActionMessage("error.customer.name.empty", lineNumber));
                        hasErrors = true; // Đánh dấu có lỗi
                    }
                    // Kiểm tra nếu email lớn hơn 40 ký tự
                    if (email.length() > 40) {
                        errors.add("email", new ActionMessage("error.email.tooLong", lineNumber));
                        hasErrors = true; // Đánh dấu có lỗi
                    }

                    // Nếu không có lỗi, kiểm tra customerId
                    if (!hasErrors) {
                        if (!customerIdStr.isEmpty()) {
                            int customerId = Integer.parseInt(customerIdStr);
                            // Kiểm tra sự tồn tại của customerId trong cơ sở dữ liệu
                            if (!isCustomerExists(customerId)) {
                                // Nếu deleteYmd khác null, thêm thông báo lỗi
                                errors.add("customerIdNotExists", new ActionMessage("error.customer.not.exists", lineNumber, customerId));
                                hasErrors = true; // Đánh dấu có lỗi
                            } else {
                                customer = new CustomerDto(customerId, customerName, sex, birthday, email, address);
                            }
                        } else {
                            // Tạo CustomerDto không bao gồm customerId
                            customer = new CustomerDto(customerName, sex, birthday, email, address);
                        }
                    }

                    // Nếu không có lỗi, thêm customer vào danh sách
                    if (!hasErrors && customer != null) {
                        customers.add(customer);
                    }
                }
                lineNumber++;
            }
        } catch (Exception e) {
            e.printStackTrace(); // Ghi log hoặc xử lý lỗi khác
        } finally {
            try {
                if (reader != null) {
                    reader.close();
                }
                if (inputStream != null) {
                    inputStream.close();
                }
            } catch (Exception e) {
                e.printStackTrace(); // Xử lý lỗi đóng file
            }
        }

        return customers;
    }

    private boolean isCustomerExists(int customerId) {
        // Phương thức kiểm tra xem customerId có tồn tại trong cơ sở dữ liệu hay không
        // Giả sử bạn đã có một lớp DAO để truy vấn dữ liệu
        return searchDao.isCustomerExists(customerId);
    }
}


```
```
upload.jsp
<%@ taglib uri="http://struts.apache.org/tags-html" prefix="html" %>
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c" %>
<html>
<head>
    <title>Upload File</title>
    <style>
        .container {
            display: flex;
            justify-content: space-between;
            width: 400px;
        }
        .hidden-file-input {
            display: none;
        }
        .custom-button {
            background-color: #4CAF50;
            color: white;
            border: none;
            padding: 10px;
            cursor: pointer;
        }
    </style>
    <script>
        function triggerFileInput() {
            document.getElementById('fileInput').click();
        }
        function updateFileName() {
            var input = document.getElementById('fileInput');
            var fileName = input.files[0] ? input.files[0].name : '';
            document.getElementById('fileName').value = fileName;
        }
        window.onload = function() {
            setTimeout(function() {
                var errorMessages = document.getElementById("htmlErrors").innerText.trim();
                if (errorMessages) {
                	alert(errorMessages.replace(/\\n/g, '\n'));
                }
            }, 100); // Thời gian chờ 2000ms (2 giây)
        }
    </script>
</head>
<body>
    <h2>Upload File Example</h2>
    <p id="messageParagraph">Hahahahea</p>
    <p id="testP"></p>
    <html:errors/>
    <h2>Example</h2>
    <!-- Thẻ chứa html:errors nhưng ẩn đi -->
    <div id="htmlErrors" style="display:none;">
        <html:errors />
    </div>
    <html:form action="/upload" enctype="multipart/form-data" onsubmit="showParagraphContent(); return false;">
        <div class="container">
            <input type="text" id="fileName" name="fileName" value="<%= request.getAttribute("fileName") != null ? request.getAttribute("fileName").toString() : "" %>" readonly="true" />
            <input type="file" id="fileInput" class="hidden-file-input" name="file" onchange="updateFileName()" />
            <button type="button" class="custom-button" onclick="triggerFileInput()">Browse</button>
        </div>
        <br/>
        <button type="submit" value="Upload" name="action">Upload</button>
    </html:form>
</body>
</html>

```
-----------------------------------------------updatene--------------------------------------------------
```
package fjs.cs.action;

import java.text.ParseException;
import java.text.SimpleDateFormat;
import java.util.Date;
import java.util.List;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import javax.servlet.http.HttpSession;

import org.apache.struts.action.Action;
import org.apache.struts.action.ActionForm;
import org.apache.struts.action.ActionForward;
import org.apache.struts.action.ActionMapping;
import org.apache.struts.action.ActionMessage;
import org.apache.struts.action.ActionMessages;

import fjs.cs.dao.SearchDao;
import fjs.cs.dto.CustomerDto;
import fjs.cs.form.SearchForm;

public class SearchAction extends Action {
    
    private SearchDao searchDao;
	
    public void setSearchDao(SearchDao searchDao) {
        this.searchDao = searchDao;
    }
//    private SearchLogic searchLogic;
//	
//    public void setSearchLogic(SearchLogic searchLogic) {
//        this.searchLogic = searchLogic;
//    }
	public ActionForward execute(ActionMapping mapping, ActionForm form,
            HttpServletRequest request, HttpServletResponse response) {
    	HttpSession session = request.getSession();
    	SearchForm searchForm = (SearchForm) form;
    	String customerName = searchForm.getCustomerName();
        String sex = searchForm.getSex();
        String fromBirthday = searchForm.getBirthdayFrom();
        String toBirthday = searchForm.getBirthdayTo();
        ActionMessages errors = new ActionMessages();
        
        String action = request.getParameter("action");
    	List<CustomerDto> customerList;
//    	if ("export".equals(action)) {
//    	    customerList = searchDao.searchCustomers(customerName, sex, fromBirthday, toBirthday);
//    	    
//    	    // Tạo file CSV
//    	    StringBuilder csvData = new StringBuilder();
//    	    
//    	    // Tiêu đề của các cột
//    	    csvData.append("\"Customer ID\",\"Name\",\"Sex\",\"Birthday\",\"Address\",\"Email\"\n");
//    	    
//    	    // Thêm dữ liệu khách hàng vào CSV, với các giá trị được bao quanh bởi dấu ngoặc kép
//    	    for (CustomerDto customer : customerList) {
//    	        csvData.append("\"").append(customer.getCustomerID()).append("\"").append(",")
//    	               .append("\"").append(customer.getCustomerName()).append("\"").append(",")
//    	               .append("\"").append(customer.getSex() == "0" ? "Female" : "Male").append("\"").append(",")
//    	               .append("\"").append(customer.getBirthday()).append("\"").append(",")
//    	               .append("\"").append(customer.getAddress()).append("\"").append(",")
//    	               .append("\"").append(customer.getEmail()).append("\"").append("\n");
//    	    }
//    	    
//    	    // Đường dẫn đến nơi lưu file CSV
//    	    String filePath = "C:/Users/ASUS/Desktop/Data/customer_export.csv";
//    	    
//    	    try {
//    	        // Ghi dữ liệu CSV vào file trên ổ đĩa
//    	        java.nio.file.Files.write(java.nio.file.Paths.get(filePath), csvData.toString().getBytes());
//    	        System.out.println("File exported successfully to " + filePath);
//    	    } catch (Exception e) {
//    	        e.printStackTrace();
//    	    }
//    	    
//    	    // Chuyển hướng hoặc thông báo thành công (có thể thêm redirect hoặc thông báo trong giao diện)
//    	    return mapping.findForward("search");
//    	}
    	if ("export".equals(action)) {
    	    customerList = searchDao.searchCustomers(customerName, sex, fromBirthday, toBirthday);
    	    
    	    // Tạo file CSV
    	    StringBuilder csvData = new StringBuilder();
    	    
    	    // Tiêu đề của các cột
    	    csvData.append("\"Customer ID\",\"Name\",\"Sex\",\"Birthday\",\"Address\",\"Email\"\n");
    	    
    	    // Thêm dữ liệu khách hàng vào CSV
    	    for (CustomerDto customer : customerList) {
    	        csvData.append("\"").append(customer.getCustomerID()).append("\",")
    	               .append("\"").append(customer.getCustomerName()).append("\",")
    	               .append("\"").append(customer.getSex() == "0" ? "Female" : "Male").append("\",")
    	               .append("\"").append(customer.getBirthday()).append("\",")
    	               .append("\"").append(customer.getAddress()).append("\",")
    	               .append("\"").append(customer.getEmail()).append("\"\n");
    	    }
    	    
    	    // Đường dẫn đến nơi lưu file CSV
    	    String filePath = "C:/Users/ASUS/Desktop/Data/customer_export.csv";
    	    
    	    try {
    	        // Ghi dữ liệu CSV vào file trên ổ đĩa
    	        java.nio.file.Files.write(java.nio.file.Paths.get(filePath), csvData.toString().getBytes());
    	        System.out.println("File exported successfully to " + filePath);
    	    } catch (Exception e) {
    	        e.printStackTrace();
    	    }
    	    
    	    // Chuyển hướng hoặc thông báo thành công (có thể thêm redirect hoặc thông báo trong giao diện)
    	    return mapping.findForward("search");
    	}




    	if ("Search".equals(action)) {
    	    // Kiểm tra định dạng YYYY/MM/DD của birthdayFrom và birthdayTo
    	    String datePattern = "^\\d{4}/\\d{2}/\\d{2}$";  // Regex kiểm tra định dạng YYYY/MM/DD}
    	    
    	    // Không có lỗi, thực hiện tìm kiếm
    	    customerList = searchDao.searchCustomers(customerName, sex, fromBirthday, toBirthday);
    	    
    	    // Logic phân trang...
    	    int PAGE_SIZE = 2;
    	    String pageStr = request.getParameter("page");
    	    Integer currentPage = 1;
    	    if (pageStr != null) {
    	        currentPage = Integer.parseInt(pageStr);
    	    }
    	    if (currentPage == null || currentPage < 1) {
    	        currentPage = 1;
    	    }
    	    int startRow = (currentPage - 1) * PAGE_SIZE;
    	    int totalCustomers = customerList.size();
    	    int totalPages = (int) Math.ceil((double) totalCustomers / PAGE_SIZE);
    	    
    	    List<CustomerDto> paginatedList = customerList.subList(
    	        Math.min(startRow, totalCustomers),
    	        Math.min(startRow + PAGE_SIZE, totalCustomers)
    	    );
    	    
    	    // Lưu thông tin tìm kiếm vào session
    	    session.setAttribute("customerName", customerName);
    	    session.setAttribute("sex", sex);
    	    session.setAttribute("fromBirthday", fromBirthday);
    	    session.setAttribute("toBirthday", toBirthday);
    	    
    	    session.setAttribute("customerList", paginatedList);
    	    session.setAttribute("currentPage", currentPage);
    	    session.setAttribute("totalPages", totalPages);
    	    
    	    return mapping.findForward("search");
    	}


        if (customerName != null || sex != null || fromBirthday != null || toBirthday != null) {
            customerList = searchDao.searchCustomers(customerName, sex, fromBirthday, toBirthday);
        } else {	
            customerList = searchDao.getAllCustomers();
        }
        int PAGE_SIZE = 2;
        String pageStr = request.getParameter("page");
        Integer currentPage = (Integer) session.getAttribute("currentPage");
        if (pageStr != null) {
            currentPage = Integer.parseInt(pageStr);
        }
        if (currentPage == null || currentPage < 1 ) {
            currentPage = 1;
        }
        int startRow = (currentPage - 1) * PAGE_SIZE;
        int totalCustomers = customerList.size();
        int totalPages = (int) Math.ceil((double) totalCustomers / PAGE_SIZE);
        List<CustomerDto> paginatedList = customerList.subList(
                Math.min(startRow, totalCustomers),
                Math.min(startRow + PAGE_SIZE, totalCustomers)
        );
        session.setAttribute("customerName", customerName);
        session.setAttribute("sex", sex);
        session.setAttribute("fromBirthday", fromBirthday);
        session.setAttribute("toBirthday", toBirthday);
        session.setAttribute("customerList", paginatedList);
        session.setAttribute("currentPage", currentPage);
        session.setAttribute("totalPages", totalPages);
    	return mapping.findForward("search");
    }
}

```
```
import fjs.cs.form.SearchForm;
import org.apache.struts.action.ActionForm;
import org.apache.struts.action.ActionForward;
import org.apache.struts.action.ActionMapping;
import org.hibernate.Session;
import org.hibernate.Transaction;
import org.hibernate.query.Query;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import org.springframework.web.multipart.MultipartFile;
import java.io.BufferedReader;
import java.io.InputStreamReader;
import java.util.ArrayList;
import java.util.List;

public class SearchAction extends org.apache.struts.action.Action {
    @Override
    public ActionForward execute(ActionMapping mapping, ActionForm form,
                                 HttpServletRequest request, HttpServletResponse response) {
        SearchForm searchForm = (SearchForm) form;
        MultipartFile file = searchForm.getFile();

        // Kiểm tra file upload
        if (file == null || file.isEmpty()) {
            String alertMessage = "File import not exist";
            request.setAttribute("alertMessage", alertMessage);
            return mapping.findForward("failure");
        }

        // Kiểm tra định dạng file
        String fileName = file.getOriginalFilename();
        if (!fileName.endsWith(".csv")) {
            String alertMessage = "File import is not valid";
            request.setAttribute("alertMessage", alertMessage);
            return mapping.findForward("failure");
        }

        List<String> errorMessages = new ArrayList<>();

        try (BufferedReader reader = new BufferedReader(new InputStreamReader(file.getInputStream()))) {
            String line;
            int lineNumber = 1;

            // Bỏ qua tiêu đề
            reader.readLine();

            while ((line = reader.readLine()) != null) {
                String[] values = line.split(",");
                String customerId = values[0].trim(); // Giả sử customerId ở cột đầu tiên

                // Kiểm tra customerId không empty và không tồn tại trong bảng mstcustomer
                if (!customerId.isEmpty() && !isCustomerIdExists(customerId)) {
                    String errorMessage = String.format("Line %d: customerId = %s is not existed", lineNumber, customerId);
                    errorMessages.add(errorMessage);
                }
                lineNumber++;
            }
        } catch (Exception e) {
            // Xử lý lỗi đọc file
            String alertMessage = "Error reading file: " + e.getMessage();
            request.setAttribute("alertMessage", alertMessage);
            return mapping.findForward("failure");
        }

        // Nếu có lỗi, hiển thị tất cả các lỗi
        if (!errorMessages.isEmpty()) {
            StringBuilder allErrors = new StringBuilder();
            for (String errorMessage : errorMessages) {
                allErrors.append(errorMessage).append("\n");
            }
            request.setAttribute("alertMessage", allErrors.toString());
            return mapping.findForward("failure");
        }

        // Nếu không có lỗi, tiếp tục xử lý
        return mapping.findForward("success");
    }

    private boolean isCustomerIdExists(String customerId) {
        // Kiểm tra sự tồn tại của customerId trong bảng mstcustomer
        Session session = HibernateUtil.getSessionFactory().openSession();
        Transaction transaction = null;
        boolean exists = false;

        try {
            transaction = session.beginTransaction();
            String hql = "FROM MstCustomer WHERE customerId = :customerId AND deleteYmd IS NULL";
            Query query = session.createQuery(hql);
            query.setParameter("customerId", customerId);
            exists = !query.list().isEmpty();
            transaction.commit();
        } catch (Exception e) {
            if (transaction != null) transaction.rollback();
            e.printStackTrace();
        } finally {
            session.close();
        }
        return exists;
    }
}

```
```
<c:if test="${not empty alertMessage}">
    <script>
        alert("${alertMessage}");
    </script>
</c:if>

```
```
ai
package fjs.cs.logic;

import java.util.List;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import fjs.cs.dao.SearchDao;
import fjs.cs.dto.CustomerDto;

@Service
public class searchLogic {

    @Autowired
    private SearchDao searchDao;

    public List<CustomerDto> searchCustomers(String customerName, String sex, String fromBirthday, String toBirthday, int pageSize, int currentPage) {
        List<CustomerDto> allCustomers = searchDao.searchCustomers(customerName, sex, fromBirthday, toBirthday);

        // Logic phân trang
        int startRow = (currentPage - 1) * pageSize;
        int totalCustomers = allCustomers.size();
        int totalPages = (int) Math.ceil((double) totalCustomers / pageSize);

        List<CustomerDto> paginatedList = allCustomers.subList(
            Math.min(startRow, totalCustomers),
            Math.min(startRow + pageSize, totalCustomers)
        );

        // Set additional attributes if needed
        // paginatedList.setTotalPages(totalPages);

        return paginatedList;
    }
}

ai
```
```
delete
import java.text.SimpleDateFormat;
import java.util.Date;
import org.hibernate.Session;
import org.hibernate.Transaction;
import org.hibernate.Query;

public class CustomerDao {
    
    private SessionFactory sessionFactory;

    public void deleteCustomer(int customerID) {
        Transaction transaction = null;
        Session session = sessionFactory.openSession();
        
        try {
            // Bắt đầu transaction
            transaction = session.beginTransaction();

            // Lấy thời gian hiện tại
            SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd");
            String currentDate = sdf.format(new Date());

            // Tạo câu lệnh HQL để cập nhật delete_ymd
            String hql = "UPDATE Customer SET delete_ymd = :delete_ymd WHERE customerID = :customerID";
            Query query = session.createQuery(hql);
            query.setParameter("delete_ymd", currentDate);
            query.setParameter("customerID", customerID);

            // Thực thi câu lệnh HQL
            int result = query.executeUpdate();

            // Commit transaction nếu không có lỗi
            transaction.commit();
        } catch (Exception e) {
            if (transaction != null) {
                transaction.rollback();
            }
            e.printStackTrace();
        } finally {
            session.close();
        }
    }
}

```
```
add 
import org.hibernate.Session;
import org.hibernate.Transaction;

public class CustomerDao {
    
    private SessionFactory sessionFactory;

    public void addCustomer(CustomerDto customerDto) {
        Transaction transaction = null;
        Session session = sessionFactory.openSession();
        
        try {
            // Bắt đầu transaction
            transaction = session.beginTransaction();
            
            // Tạo đối tượng Customer từ CustomerDto
            Customer customer = new Customer();
            customer.setCustomerID(customerDto.getCustomerID());   // Nếu customerID là tự động thì bỏ qua dòng này
            customer.setCustomerName(customerDto.getCustomerName());
            customer.setSex(customerDto.getSex());
            customer.setBirthday(customerDto.getBirthday());
            customer.setAddress(customerDto.getAddress());
            customer.setDeleteYmd(null);  // Ban đầu giá trị delete_ymd là null
            
            // Lưu đối tượng vào cơ sở dữ liệu
            session.save(customer);

            // Commit transaction nếu không có lỗi
            transaction.commit();
        } catch (Exception e) {
            if (transaction != null) {
                transaction.rollback();
            }
            e.printStackTrace();
        } finally {
            session.close();
        }
    }
}

```
```
edit 
import org.hibernate.Session;
import org.hibernate.Transaction;

public class CustomerDao {

    private SessionFactory sessionFactory;

    public void editCustomer(CustomerDto customerDto) {
        Transaction transaction = null;
        Session session = sessionFactory.openSession();
        
        try {
            // Bắt đầu transaction
            transaction = session.beginTransaction();

            // Truy xuất khách hàng từ database dựa trên customerID
            Customer customer = (Customer) session.get(Customer.class, customerDto.getCustomerID());
            
            // Kiểm tra nếu khách hàng tồn tại
            if (customer != null) {
                // Cập nhật thông tin từ customerDto
                customer.setCustomerName(customerDto.getCustomerName());
                customer.setSex(customerDto.getSex());
                customer.setBirthday(customerDto.getBirthday());
                customer.setAddress(customerDto.getAddress());
                // Không chỉnh sửa delete_ymd vì chỉ được thay đổi khi xóa
                
                // Cập nhật đối tượng trong database
                session.update(customer);

                // Commit transaction sau khi cập nhật
                transaction.commit();
            } else {
                System.out.println("Customer with ID " + customerDto.getCustomerID() + " not found.");
            }
        } catch (Exception e) {
            if (transaction != null) {
                transaction.rollback();
            }
            e.printStackTrace();
        } finally {
            session.close();
        }
    }
}

```
```
import org.hibernate.Session;
import org.hibernate.Transaction;
import java.util.List;

public class CustomerDao {

    private SessionFactory sessionFactory;

    @SuppressWarnings("unchecked")
    public List<Object[]> getAllCustomerNamesAndAddresses() {
        Transaction transaction = null;
        List<Object[]> customerList = null;
        Session session = sessionFactory.openSession();
        
        try {
            // Bắt đầu transaction
            transaction = session.beginTransaction();

            // Viết câu HQL để lấy customerName và address
            String hql = "SELECT c.customerName, c.address FROM Customer c WHERE c.delete_ymd IS NULL";
            Query query = session.createQuery(hql);
            
            // Thực thi query và lấy danh sách kết quả
            customerList = query.list();
            
            // Commit transaction
            transaction.commit();
        } catch (Exception e) {
            if (transaction != null) {
                transaction.rollback();
            }
            e.printStackTrace();
        } finally {
            session.close();
        }
        
        return customerList;
    }
}

```

```
begin
package fjs.cs.action;

import java.text.ParseException;
import java.text.SimpleDateFormat;
import java.util.Date;
import java.util.List;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import javax.servlet.http.HttpSession;

import org.apache.struts.action.Action;
import org.apache.struts.action.ActionForm;
import org.apache.struts.action.ActionForward;
import org.apache.struts.action.ActionMapping;
import org.apache.struts.action.ActionMessage;
import org.apache.struts.action.ActionMessages;

import fjs.cs.dao.SearchDao;
import fjs.cs.dto.CustomerDto;
import fjs.cs.form.SearchForm;

public class SearchAction extends Action {
    
    private SearchDao searchDao;
	
    public void setSearchDao(SearchDao searchDao) {
        this.searchDao = searchDao;
    }
//    private SearchLogic searchLogic;
//	
//    public void setSearchLogic(SearchLogic searchLogic) {
//        this.searchLogic = searchLogic;
//    }
	public ActionForward execute(ActionMapping mapping, ActionForm form,
            HttpServletRequest request, HttpServletResponse response) {
    	HttpSession session = request.getSession();
    	SearchForm searchForm = (SearchForm) form;
    	String customerName = searchForm.getCustomerName();
        String sex = searchForm.getSex();
        String fromBirthday = searchForm.getBirthdayFrom();
        String toBirthday = searchForm.getBirthdayTo();
        ActionMessages errors = new ActionMessages();
        
        String action = request.getParameter("action");
    	List<CustomerDto> customerList;
    	if ("Search".equals(action)) {
    	    // Kiểm tra định dạng YYYY/MM/DD của birthdayFrom và birthdayTo
    	    String datePattern = "^\\d{4}/\\d{2}/\\d{2}$";  // Regex kiểm tra định dạng YYYY/MM/DD}
    	    
    	    // Không có lỗi, thực hiện tìm kiếm
    	    customerList = searchDao.searchCustomers(customerName, sex, fromBirthday, toBirthday);
    	    
    	    // Logic phân trang...
    	    int PAGE_SIZE = 2;
    	    String pageStr = request.getParameter("page");
    	    Integer currentPage = 1;
    	    if (pageStr != null) {
    	        currentPage = Integer.parseInt(pageStr);
    	    }
    	    if (currentPage == null || currentPage < 1) {
    	        currentPage = 1;
    	    }
    	    int startRow = (currentPage - 1) * PAGE_SIZE;
    	    int totalCustomers = customerList.size();
    	    int totalPages = (int) Math.ceil((double) totalCustomers / PAGE_SIZE);
    	    
    	    List<CustomerDto> paginatedList = customerList.subList(
    	        Math.min(startRow, totalCustomers),
    	        Math.min(startRow + PAGE_SIZE, totalCustomers)
    	    );
    	    
    	    // Lưu thông tin tìm kiếm vào session
    	    session.setAttribute("customerName", customerName);
    	    session.setAttribute("sex", sex);
    	    session.setAttribute("fromBirthday", fromBirthday);
    	    session.setAttribute("toBirthday", toBirthday);
    	    
    	    session.setAttribute("customerList", paginatedList);
    	    session.setAttribute("currentPage", currentPage);
    	    session.setAttribute("totalPages", totalPages);
    	    
    	    return mapping.findForward("search");
    	}


        if (customerName != null || sex != null || fromBirthday != null || toBirthday != null) {
            customerList = searchDao.searchCustomers(customerName, sex, fromBirthday, toBirthday);
        } else {	
            customerList = searchDao.getAllCustomers();
        }
        int PAGE_SIZE = 2;
        String pageStr = request.getParameter("page");
        Integer currentPage = (Integer) session.getAttribute("currentPage");
        if (pageStr != null) {
            currentPage = Integer.parseInt(pageStr);
        }
        if (currentPage == null || currentPage < 1 ) {
            currentPage = 1;
        }
        int startRow = (currentPage - 1) * PAGE_SIZE;
        int totalCustomers = customerList.size();
        int totalPages = (int) Math.ceil((double) totalCustomers / PAGE_SIZE);
        List<CustomerDto> paginatedList = customerList.subList(
                Math.min(startRow, totalCustomers),
                Math.min(startRow + PAGE_SIZE, totalCustomers)
        );
        session.setAttribute("customerName", customerName);
        session.setAttribute("sex", sex);
        session.setAttribute("fromBirthday", fromBirthday);
        session.setAttribute("toBirthday", toBirthday);
        session.setAttribute("customerList", paginatedList);
        session.setAttribute("currentPage", currentPage);
        session.setAttribute("totalPages", totalPages);
    	return mapping.findForward("search");
    }
}

```
```
SEARCHDAO
package fjs.cs.dao;

import java.util.ArrayList;
import java.util.List;

import org.hibernate.Criteria;
import org.hibernate.Query;
import org.hibernate.Session;
import org.hibernate.SessionFactory;
import org.hibernate.Transaction;

import fjs.cs.dto.CustomerDto;
import fjs.cs.model.Customer;

public class SearchDao {
	
	private SessionFactory sessionFactory;


	public void setSessionFactory(SessionFactory sessionFactory) {
	   this.sessionFactory = sessionFactory;
	}
	@SuppressWarnings("unchecked")
	public List<CustomerDto> getAllCustomers() {
	    Session session = sessionFactory.openSession();
	    String hql = "SELECT c.customerID, c.customerName, c.sex, c.birthday, c.address FROM Customer c WHERE c.delete_ymd IS NULL";
	    List<Object[]> results = session.createQuery(hql).list();
	    session.close();
	
	    List<CustomerDto> customers = new ArrayList<>();
	    for (Object[] row : results) {
	        CustomerDto dto = new CustomerDto();
	        dto.setCustomerID((int) row[0]);
	        dto.setCustomerName((String) row[1]);
	        dto.setSex((String) row[2]);
	        dto.setBirthday((String) row[3]);
	        dto.setAddress((String) row[4]);
	        customers.add(dto);
	    }
	
	    return customers;
	}		
//	@SuppressWarnings("unchecked")
//	public List<CustomerDto> getAllCustomers() {
//    	Transaction transaction = null;
//    	List<CustomerDto> customer = new ArrayList<>();
//        Session session = sessionFactory.openSession();
//        transaction = session.beginTransaction();
//        String hql = "SELECT new fjs.cs.dto.CustomerDto(c.customerID, c.customerName, c.sex, c.birthday, c.address) " +
//                "FROM Customer c WHERE c.delete_ymd IS NULL";
//        Query query = session.createQuery(hql);
//        customer  = query.list();
//        transaction.commit();
//        return customer;
//    }

	@SuppressWarnings("unchecked")
	public List<CustomerDto> searchCustomers(String customerName, String sex, String birthdayFrom, String birthdayTo) {

	    Session session = sessionFactory.openSession();
	    
	    try {
	        // Tạo câu truy vấn HQL
	        StringBuilder hql = new StringBuilder("SELECT c.customerID, c.customerName, c.sex, c.birthday, c.address FROM Customer c WHERE c.delete_ymd IS NULL");
	        List<Object> params = new ArrayList<>();

	        // Thêm điều kiện tìm kiếm
	        if (customerName != null && !customerName.trim().isEmpty()) {
	            hql.append(" AND c.customerName LIKE ?");
	            params.add("%" + customerName + "%");
	        }
	        
	        if (sex != null && !sex.trim().isEmpty()) {
	            hql.append(" AND c.sex = ?");
	            params.add(sex);
	        }
	        
	        if (birthdayFrom != null && !birthdayFrom.trim().isEmpty()) {
	            hql.append(" AND c.birthday >= ?");
	            params.add(birthdayFrom);
	        }
	        
	        if (birthdayTo != null && !birthdayTo.trim().isEmpty()) {
	            hql.append(" AND c.birthday <= ?");
	            params.add(birthdayTo);
	        }

	        // Thực thi query
	        Query query = session.createQuery(hql.toString());
	        for (int i = 0; i < params.size(); i++) {
	            query.setParameter(i, params.get(i));
	        }

	        // Lấy danh sách kết quả trả về
	        List<Object[]> results = query.list();  // Kết quả là danh sách Object[]
	        List<CustomerDto> customerDtos = new ArrayList<>();

	        // Xử lý kết quả trả về
	        for (Object[] row : results) {
	            CustomerDto dto = new CustomerDto(
	                (Integer) row[0],   // customerID
	                (String) row[1],    // customerName
	                (String) row[2],    // sex
	                (String) row[3],    // birthday
	                (String) row[4]     // address
	            );
	            customerDtos.add(dto);
	        }
	        
	        return customerDtos;
	    } finally {
	        session.close();
	    }
	}		
}

```
```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE hibernate-mapping PUBLIC 
    "-//Hibernate/Hibernate Mapping DTD 3.0//EN"
    "http://www.hibernate.org/dtd/hibernate-mapping-3.0.dtd">
<hibernate-mapping>
	<class name="fjs.cs.model.Customer" table="MSTCUSTOMERS">
	    <id name="customerID" column="CUSTOMERID" type="int">
	    <generator class="native"/>
	    </id>
		<property name="customerName" column="CUSTOMERNAME" type="string" length="50"/>
		<property name="sex" column="SEX" type="string" length="50"/>
		<property name="birthday" column="BIRTHDAY" type="string" length="50"/>
		<property name="address" column="ADDRESS" type="string" length="50"/>
		<property name="email" column="EMAIL" type="string" length="50"/>
		<property name="insert_psn" column="INSERT_PSN" type="int" length="50"/>
		<property name="update_psn" column="UPDATE_PSN" type="int" length="50"/>
		<property name="delete_ymd" column="DELETE_YMD" type="timestamp" />
		<property name="insert_ymd" column="INSERT_YMD" type="timestamp" />
		<property name="update_ymd" column="UPDATE_YMD" type="timestamp" />
	</class>
</hibernate-mapping>
```
```
package fjs.cs.model;

import java.sql.Date;

public class Customer {
	private int customerID;
	private String customerName;
	private String sex;
	private String birthday;
	private String address;
	private String email;
	private int insert_psn;
	private int update_psn;
	private Date delete_ymd;
	private Date insert_ymd;
	private Date update_ymd;
	
	public Customer() {
		super();
	}
	public Customer(int customerID, String customerName, String sex, String birthday, String address) {
		super();
		this.customerID = customerID;
		this.customerName = customerName;
		this.sex = sex;
		this.birthday = birthday;
		this.address = address;
	}
	public int getCustomerID() {
		return customerID;
	}
	public void setCustomerID(int customerID) {
		this.customerID = customerID;
	}
	public String getCustomerName() {
		return customerName;
	}
	public void setCustomerName(String customerName) {
		this.customerName = customerName;
	}
	public String getSex() {
		return sex;
	}
	public void setSex(String sex) {
		this.sex = sex;
	}
	public String getBirthday() {
		return birthday;
	}
	public void setBirthday(String birthday) {
		this.birthday = birthday;
	}
	public String getAddress() {
		return address;
	}
	public void setAddress(String address) {
		this.address = address;
	}
	public String getEmail() {
		return email;
	}
	public void setEmail(String email) {
		this.email = email;
	}
	public int getInsert_psn() {
		return insert_psn;
	}
	public void setInsert_psn(int insert_psn) {
		this.insert_psn = insert_psn;
	}
	public int getUpdate_psn() {
		return update_psn;
	}
	public void setUpdate_psn(int update_psn) {
		this.update_psn = update_psn;
	}
	public Date getDelete_ymd() {
		return delete_ymd;
	}
	public void setDelete_ymd(Date delete_ymd) {
		this.delete_ymd = delete_ymd;
	}
	public Date getInsert_ymd() {
		return insert_ymd;
	}
	public void setInsert_ymd(Date insert_ymd) {
		this.insert_ymd = insert_ymd;
	}
	public Date getUpdate_ymd() {
		return update_ymd;
	}
	public void setUpdate_ymd(Date update_ymd) {
		this.update_ymd = update_ymd;
	}
	
	
}

```
```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE hibernate-configuration PUBLIC
        "-//Hibernate/Hibernate Configuration DTD 3.0//EN"
        "http://www.hibernate.org/dtd/hibernate-configuration-3.0.dtd">
<hibernate-configuration>
    <session-factory>
        <!-- Database connection settings -->
        <property name="hibernate.connection.driver_class">com.mysql.jdbc.Driver</property>
        <property name="hibernate.connection.url">jdbc:mysql://localhost:3306/CustomerSystem</property>
        <property name="hibernate.connection.username">root</property>
        <property name="hibernate.connection.password">0973129264a</property>

        <!-- JDBC connection pool settings -->
        <property name="hibernate.c3p0.min_size">5</property>
        <property name="hibernate.c3p0.max_size">20</property>
        <property name="hibernate.c3p0.timeout">300</property>
        <property name="hibernate.c3p0.max_statements">50</property>
        <property name="hibernate.c3p0.idle_test_period">3000</property>

        <!-- Specify dialect -->
        <property name="hibernate.dialect">org.hibernate.dialect.MySQLDialect</property>

        <!-- Echo all executed SQL to stdout -->
        <property name="hibernate.show_sql">true</property>

        <!-- Drop and re-create the database schema on startup -->
        <property name="hibernate.hbm2ddl.auto">update</property>

        <!-- Names the annotated entity class -->
        <mapping resource="fjs/cs/model/Login.hbm.xml"/>
        <mapping resource="fjs/cs/model/Customer.hbm.xml"/>
    </session-factory>
</hibernate-configuration>
```
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
           http://www.springframework.org/schema/beans/spring-beans.xsd">

    <!-- Cấu hình datasource kết nối MySQL -->
    <bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
        <property name="driverClassName" value="com.mysql.cj.jdbc.Driver" />
        <property name="url" value="jdbc:mysql://localhost:3306/Customersystem"/>
        <property name="username" value="root" />
        <property name="password" value="0973129264a" />
    </bean>

    <bean id="sessionFactory"
        class="org.springframework.orm.hibernate3.LocalSessionFactoryBean">
        <property name="dataSource" ref="dataSource" />
        <property name="configLocation" value="classpath:hibernate.cfg.xml" />
    </bean>
    <bean id="transactionManager" class="org.springframework.orm.hibernate3.HibernateTransactionManager">
    	<property name="sessionFactory" ref="sessionFactory" />
	</bean>
    
    <bean id="loginDao" class="fjs.cs.dao.loginDao">
    	<property name="sessionFactory" ref="sessionFactory"/>
	</bean>
	<bean id="loginLogic" class="fjs.cs.logic.loginLogic">
    	<property name="loginDao" ref="loginDao"/>
	</bean>
	<bean name= "/login" id="loginAction" class="fjs.cs.action.loginAction">
	    <property name="loginLogic" ref="loginLogic"/>
	</bean>
	<bean id="searchDao" class="fjs.cs.dao.SearchDao">
    	<property name="sessionFactory" ref="sessionFactory"/>
	</bean>
	<bean id="searchLogic" class="fjs.cs.logic.SearchLogic">
    	<property name="searchDao" ref="searchDao"/>
	</bean>
	<bean name= "/search" id="SearchAction" class="fjs.cs.action.SearchAction">
		<property name="searchLogic" ref="searchLogic"/>
	</bean>
	
    
</beans>

```
```
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE struts-config PUBLIC
    "-//Apache Software Foundation//DTD Struts Configuration 1.3//EN"
    "dtd/struts-config_1_3.dtd">

<struts-config>
	<form-beans>
	    <form-bean name="loginForm" type="fjs.cs.form.loginForm"/>
	    <form-bean name="searchForm" type="fjs.cs.form.SearchForm"/>
	</form-beans>
	
	<action-mappings>
	    <action path="/login"
	            type="org.springframework.web.struts.DelegatingActionProxy"
	            name="loginForm"
	            scope="request"
	            validate="false">
	      <forward name="success" path="/search.do" redirect="true"/>
	      <forward name="failure" path="/jsp/login.jsp" />
	    </action>
	    <action path="/search"
	            type="org.springframework.web.struts.DelegatingActionProxy"
	            name="searchForm"
	            scope="request"
	            validate="true">
	        <forward name="search" path="/jsp/Search.jsp" />
	        <forward name="login" path="/login.do" redirect="true"/>
	    </action>
	    
	</action-mappings>
	<controller processorClass="org.springframework.web.struts.DelegatingRequestProcessor"/>
	<message-resources parameter="message" />
</struts-config>

```
```
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns="http://xmlns.jcp.org/xml/ns/javaee" xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd" id="WebApp_ID" version="4.0">
  	<display-name>CustomerSystem_Struts</display-name>
	<context-param>
	    <param-name>javax.servlet.jsp.jstl.fmt.encoding</param-name>
	    <param-value>UTF-8</param-value>
	</context-param>
	    <!-- Spring Context Configuration -->
    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>/WEB-INF/spring/applicationContext.xml</param-value>
    </context-param>
    
    <!-- Spring Context Loader Listener -->
    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>

	<welcome-file-list>
	    <welcome-file>index.jsp</welcome-file> <!-- Đặt trang login.do làm welcome file -->
	</welcome-file-list>

    <servlet>
        <servlet-name>action</servlet-name>
        <servlet-class>org.apache.struts.action.ActionServlet</servlet-class>
        <init-param>
            <param-name>config</param-name>
            <param-value>/WEB-INF/struts-config.xml</param-value>
        </init-param>
        <load-on-startup>2</load-on-startup>
    </servlet>

    <servlet-mapping>
        <servlet-name>action</servlet-name>
        <url-pattern>*.do</url-pattern>
    </servlet-mapping>
</web-app>
```

DAY 5 --------------------------------------------


https://drive.google.com/drive/folders/1GCwNOsEnXPaDy0JoxNVx5eWhrDPhAVsL?usp=sharing
SQL SERVER
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
           http://www.springframework.org/schema/beans/spring-beans.xsd">

    <!-- Cấu hình datasource kết nối SQL Server -->
    <bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
        <property name="driverClassName" value="com.microsoft.sqlserver.jdbc.SQLServerDriver" />
        <property name="url" value="jdbc:sqlserver://localhost:1433;databaseName=CustomerSystem"/>
        <property name="username" value="sa" />
        <property name="password" value="123456" />
    </bean>

    <bean id="sessionFactory" class="org.springframework.orm.hibernate3.LocalSessionFactoryBean">
        <property name="dataSource" ref="dataSource" />
        <property name="configLocation" value="classpath:hibernate.cfg.xml" />
    </bean>

    <bean id="loginDao" class="fjs.cs.dao.loginDao">
        <property name="sessionFactory" ref="sessionFactory"/>
    </bean>

</beans>

```
```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE hibernate-configuration PUBLIC
        "-//Hibernate/Hibernate Configuration DTD 3.0//EN"
        "http://www.hibernate.org/dtd/hibernate-configuration-3.0.dtd">
<hibernate-configuration>
    <session-factory>
        <!-- Database connection settings -->
        <property name="hibernate.connection.driver_class">com.microsoft.sqlserver.jdbc.SQLServerDriver</property>
        <property name="hibernate.connection.url">jdbc:sqlserver://localhost:1433;databaseName=CustomerSystem</property>
        <property name="hibernate.connection.username">sa</property>
        <property name="hibernate.connection.password">123456</property>

        <!-- JDBC connection pool settings -->
        <property name="hibernate.c3p0.min_size">5</property>
        <property name="hibernate.c3p0.max_size">20</property>
        <property name="hibernate.c3p0.timeout">300</property>
        <property name="hibernate.c3p0.max_statements">50</property>
        <property name="hibernate.c3p0.idle_test_period">3000</property>

        <!-- Specify dialect for SQL Server -->
        <property name="hibernate.dialect">org.hibernate.dialect.SQLServerDialect</property>

        <!-- Echo all executed SQL to stdout -->
        <property name="hibernate.show_sql">true</property>

        <!-- Auto update schema -->
        <property name="hibernate.hbm2ddl.auto">update</property>

        <!-- Names the annotated entity class -->
        <mapping resource="fjs/cs/model/Login.hbm.xml"/>
    </session-factory>
</hibernate-configuration>
```
SQL SERVER 
```
loginAction.java
package fjs.cs.action;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import org.apache.struts.action.Action;
import org.apache.struts.action.ActionForm;
import org.apache.struts.action.ActionForward;
import org.apache.struts.action.ActionMapping;
import org.apache.struts.action.ActionServlet;
import org.springframework.web.context.WebApplicationContext;
import org.springframework.web.context.support.WebApplicationContextUtils;
import fjs.cs.dao.loginDao;
import fjs.cs.form.loginForm;

public class loginAction extends Action {

    private loginDao loginDao;

    @Override
    public void setServlet(ActionServlet servlet) {
        super.setServlet(servlet);
        WebApplicationContext context = WebApplicationContextUtils.getRequiredWebApplicationContext(servlet.getServletContext());
        this.loginDao = (loginDao) context.getBean("loginDao");
    }

    @Override
    public ActionForward execute(ActionMapping mapping, ActionForm form,
                                 HttpServletRequest request, HttpServletResponse response) {
        loginForm loginForm = (loginForm) form;

        // Lấy thông tin người dùng từ form
        String userName = loginForm.getUserID();
        String passWord = loginForm.getPassWord();

        // Kiểm tra thông tin đăng nhập
        int count = loginDao.checkLogin(userName, passWord);

        if (count > 0) {
            // Đăng nhập thành công
            return mapping.findForward("success");
        } else {
            // Đăng nhập thất bại
            request.setAttribute("errorMessage", "Invalid username or password.");
            return mapping.findForward("failure");
        }
    }
}
```
```
loginDao
package fjs.cs.dao;

import java.util.List;

import org.hibernate.Session;
import org.hibernate.SessionFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Repository;


public class loginDao {
	

	private SessionFactory sessionFactory;


	public void setSessionFactory(SessionFactory sessionFactory) {
	   this.sessionFactory = sessionFactory;
	}

    public int checkLogin(String userName, String passWord) {
        Session session = sessionFactory.openSession();
        System.out.println(userName);
        String hql = "SELECT COUNT(*) FROM Login l WHERE l.deleteYMD IS NULL AND l.userID = :userID AND l.passWord = :passWord";
        
        Long count = (Long) session.createQuery(hql)
                                  .setParameter("userID", userName)
                                  .setParameter("passWord", passWord)
                                  .uniqueResult();
        session.close();
        System.out.println(count.intValue());
        return count.intValue();
    }
}
```
```
loginForm.java
package fjs.cs.form;

import org.apache.struts.action.ActionForm;

public class loginForm extends ActionForm {
	/**
	 * 
	 */
	private static final long serialVersionUID = 1L;
	private String userID; 
	private String passWord;
	
	public loginForm() {
		super();
	}

	public String getUserID() {
		return userID;
	}

	public void setUserID(String userID) {
		this.userID = userID;
	}

	public String getPassWord() {
		return passWord;
	}

	public void setPassWord(String passWord) {
		this.passWord = passWord;
	} 
}

```
```
Login.java (model)
package fjs.cs.model;

public class Login {
	private int psn_cd; 
	private String userID;
	private String userName;
	private String passWord;
	private String deleteYMD;
	public int getPsn_cd() {
		return psn_cd;
	}
	public void setPsn_cd(int psn_cd) {
		this.psn_cd = psn_cd;
	}
	public String getUserID() {
		return userID;
	}
	public void setUserID(String userID) {
		this.userID = userID;
	}
	public String getUserName() {
		return userName;
	}
	public void setUserName(String userName) {
		this.userName = userName;
	}
	public String getPassWord() {
		return passWord;
	}
	public void setPassWord(String passWord) {
		this.passWord = passWord;
	}
	public String getDeleteYMD() {
		return deleteYMD;
	}
	public void setDeleteYMD(String deleteYMD) {
		this.deleteYMD = deleteYMD;
	}
	
	
}
```
```
Login.hbm.xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE hibernate-mapping PUBLIC 
    "-//Hibernate/Hibernate Mapping DTD 3.0//EN"
    "http://www.hibernate.org/dtd/hibernate-mapping-3.0.dtd">

<hibernate-mapping>
    <class name="fjs.cs.model.Login" table="MSTUSERS">
        <id name="psn_cd" column="PSN_CD" type="int">
        </id>
        <property name="userID" column="USERID" type="string" length="50"/>
        <property name="userName" column="USERNAME" type="string" length="100"/>
        <property name="passWord" column="PASSWORD" type="string" length="100"/>
        <property name="deleteYMD" column="DELETE_YMD" type="timestamp"/>
    </class>
</hibernate-mapping>
```
```
hibernate.cfg.xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE hibernate-configuration PUBLIC
        "-//Hibernate/Hibernate Configuration DTD 3.0//EN"
        "http://www.hibernate.org/dtd/hibernate-configuration-3.0.dtd">
<hibernate-configuration>
    <session-factory>
        <!-- Database connection settings -->
        <property name="hibernate.connection.driver_class">com.mysql.jdbc.Driver</property>
        <property name="hibernate.connection.url">jdbc:mysql://localhost:3306/CustomerSystem</property>
        <property name="hibernate.connection.username">root</property>
        <property name="hibernate.connection.password">0973129264a</property>

        <!-- JDBC connection pool settings -->
        <property name="hibernate.c3p0.min_size">5</property>
        <property name="hibernate.c3p0.max_size">20</property>
        <property name="hibernate.c3p0.timeout">300</property>
        <property name="hibernate.c3p0.max_statements">50</property>
        <property name="hibernate.c3p0.idle_test_period">3000</property>

        <!-- Specify dialect -->
        <property name="hibernate.dialect">org.hibernate.dialect.MySQLDialect</property>

        <!-- Echo all executed SQL to stdout -->
        <property name="hibernate.show_sql">true</property>

        <!-- Drop and re-create the database schema on startup -->
        <property name="hibernate.hbm2ddl.auto">update</property>

        <!-- Names the annotated entity class -->
        <mapping resource="fjs/cs/model/Login.hbm.xml"/>
    </session-factory>
</hibernate-configuration>
```
```
login.jsp
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c" %> 
<%@ taglib uri="http://struts.apache.org/tags-html" prefix="html" %>
<%@ taglib uri="/WEB-INF/struts-bean.tld" prefix="bean" %>
<%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8"%>
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<style type="text/css">
	body {
		margin-left :20px;
		margin-right : 20px;
		background-color : #87CEFA;	
	}
	.header {
		color : red;
		border-bottom : 2px solid;
		
	}
    .login-container {
        display: flex;
        justify-content: center;
    }

    .login-form {
        display: flex;
        flex-direction: column;
        padding: 20px;
        border-radius: 10px;
        box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
        width: 300px; /* Set a width for the form */
    }

    .form-title {
        font-size: 24px;
        margin-bottom: 20px;
        text-align: center;
        color : blue;
    }

    .form-group {
        display: flex;
        justify-content: space-between;
        align-items: center;
        margin-bottom: 15px;
    }

    .form-group label {
        margin-right: 10px;
        width: 80px; /* Set a fixed width for labels */
    }

    .form-group input {
        width: 70%;
        padding: 5px;
        border: 1px solid #ccc;
        border-radius: 2px;
    }

    .login-form input[type="submit"],
    .login-form input[type="button"] {
        width: 120px;
        margin: 5px;
        cursor: pointer;
    }
	
    #errorMessageDiv {
        margin-bottom: 15px;
        color: red;
        text-align: center;
        min-height: 20px; /* de form khong bi day xuong khi hien ra  */
    }
    .btnDiv {
        display: flex;
        justify-content: center;
    }
</style>
<title>Login</title>
</head>
<body>
	<div class= header>
		<h1>TRAINING</h1>
	</div>				
	<div class="container">
		<div class="login-container">
		    <div class="login-form">
		        <div class="form-title">Login</div> 
		        <div id="errorMessageDiv">
		            <html:errors />
		        </div>
		        <html:form action="/login" method="post">
		            <div class="form-group">
		                <label for="username">Username:</label>
		                <html:text property="userID" />
		            </div>
		            <div class="form-group">
		                <label for="passWord">Password:</label>
		                <html:password property="passWord" />
		            </div>
		            <div class= "btnDiv">
		            	<html:submit value= "Login" property= "action"/>
		            	<html:submit value="Clear" property="action" onclick="clearForm()" />
		            </div>
		        </html:form>
		    </div>
		</div>
	</div>
</body>
</html>
```
```
applicationContext
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
           http://www.springframework.org/schema/beans/spring-beans.xsd">

    <!-- Cấu hình datasource kết nối MySQL -->
    <bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
        <property name="driverClassName" value="com.mysql.cj.jdbc.Driver" />
        <property name="url" value="jdbc:mysql://localhost:3306/Customersystem"/>
        <property name="username" value="root" />
        <property name="password" value="0973129264a" />
    </bean>

    <bean id="sessionFactory"
        class="org.springframework.orm.hibernate3.LocalSessionFactoryBean">
        <property name="dataSource" ref="dataSource" />
        <property name="configLocation" value="classpath:hibernate.cfg.xml" />
    </bean>
    <bean id="loginDao" class="fjs.cs.dao.loginDao">
    	<property name="sessionFactory" ref="sessionFactory"/>
	</bean>

    
</beans>
```
```
struts-config
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE struts-config PUBLIC
    "-//Apache Software Foundation//DTD Struts Configuration 1.3//EN"
    "dtd/struts-config_1_3.dtd">

<struts-config>
	<form-beans>
	    <form-bean name="loginForm" type="fjs.cs.form.loginForm"/>
	</form-beans>
	
	<action-mappings>
	    <action path="/login"
	            type="fjs.cs.action.loginAction"
	            name="loginForm"
	            scope="request"
	            validate="true">
	        <forward name="success" path="/search.do" redirect="true"/>
	        <forward name="failure" path="/jsp/login.jsp" />
	    </action>
	    
	</action-mappings>
	<message-resources parameter="message" />
</struts-config>
```
```
web.xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE web-app PUBLIC 
    "-//Sun Microsystems, Inc.//DTD Web Application 2.3//EN" 
    "http://java.sun.com/dtd/web-app_2_3.dtd">

<web-app>
    <display-name>HelloStruts1x</display-name>

    <!-- Spring Context Configuration -->
    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>/WEB-INF/spring/applicationContext.xml</param-value>
    </context-param>
    
    <!-- Spring Context Loader Listener -->
    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>

    <!-- Struts Action Servlet -->
    <servlet>
        <servlet-name>action</servlet-name>
        <servlet-class>org.apache.struts.action.ActionServlet</servlet-class>
        <init-param>
            <param-name>config</param-name>
            <param-value>/WEB-INF/struts-config.xml</param-value>
        </init-param>
        <load-on-startup>2</load-on-startup>
    </servlet>

    <servlet-mapping>
        <servlet-name>action</servlet-name>
        <url-pattern>*.do</url-pattern>
    </servlet-mapping>
    
    <welcome-file-list>
        <welcome-file>index.jsp</welcome-file>
    </welcome-file-list>
</web-app>
```








-------------------------TOI DAY LA HETTTT ---------------------------------------------------------
























```
hibernate.cfg.xml
Đây là tệp cấu hình Hibernate để kết nối với SQL Server.
<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE hibernate-configuration PUBLIC
        "-//Hibernate/Hibernate Configuration DTD 3.0//EN"
        "http://hibernate.sourceforge.net/hibernate-configuration-3.0.dtd">
<hibernate-configuration>
    <session-factory>
        <property name="hibernate.dialect">org.hibernate.dialect.SQLServerDialect</property>
        <property name="hibernate.connection.driver_class">com.microsoft.sqlserver.jdbc.SQLServerDriver</property>
        <property name="hibernate.connection.url">jdbc:sqlserver://localhost:1433;databaseName=CustomerSystem</property>
        <property name="hibernate.connection.username">CustomerSystem</property>
        <property name="hibernate.connection.password">123456</property>
        <property name="hibernate.hbm2ddl.auto">update</property>
        <property name="show_sql">true</property>
        
        <mapping resource="fjs/cs/model/MstUser.hbm.xml"/>
    </session-factory>
</hibernate-configuration>
```
```
loginDao.java
Tệp này đã được cập nhật để sử dụng Hibernate thay vì JDBC.
package fjs.cs.dao;

import org.hibernate.Session;
import org.hibernate.Transaction;
import org.hibernate.query.Query;
import fjs.cs.model.MstUser;
import fjs.cs.util.HibernateUtil;

public class loginDao {
    public int checkLogin(String userName, String passWord) {
        int cnt = 0;
        Transaction transaction = null;

        try (Session session = HibernateUtil.getSessionFactory().openSession()) {
            transaction = session.beginTransaction();

            String hql = "SELECT COUNT(*) FROM MstUser WHERE deleteYmd IS NULL AND userID = :username AND password = :password";
            Query<Long> query = session.createQuery(hql, Long.class);
            query.setParameter("username", userName);
            query.setParameter("password", passWord);

            cnt = query.uniqueResult().intValue();

            transaction.commit();
        } catch (Exception e) {
            if (transaction != null) {
                transaction.rollback();
            }
            e.printStackTrace();
        }
        return cnt;
    }
}

```
```
loginAction.java
package fjs.cs.action;

import java.sql.SQLException;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import javax.servlet.http.HttpSession;

import org.apache.struts.action.Action;
import org.apache.struts.action.ActionForm;
import org.apache.struts.action.ActionForward;
import org.apache.struts.action.ActionMapping;
import org.apache.struts.action.ActionMessage;
import org.apache.struts.action.ActionMessages;
import fjs.cs.dao.loginDao;
import fjs.cs.form.loginForm;

public class loginAction extends Action {
    public ActionForward execute(ActionMapping mapping, ActionForm form,
                                 HttpServletRequest req, HttpServletResponse resp) {
        loginForm loginForm = (loginForm) form;
        String userid = loginForm.getUserID();
        String password = loginForm.getPassWord();
        String action = req.getParameter("action");

        ActionMessages errors = new ActionMessages();
        if ("Login".equals(action)) {
            if (userid == null || userid.trim().isEmpty()) {
                errors.add("username", new ActionMessage("error.username.required"));
                saveErrors(req, errors);
                return mapping.findForward("failure");
            }
            if (password == null || password.trim().isEmpty()) {
                errors.add("password", new ActionMessage("error.password.required"));
                saveErrors(req, errors);
                return mapping.findForward("failure");
            } else {
                loginDao loginDao = new loginDao();
                int cnt = loginDao.checkLogin(userid, password);
                if (cnt == 1) {
                    HttpSession session = req.getSession();
                    session.setAttribute("userid", userid);
                    return mapping.findForward("success");
                } else {
                    errors.add("login", new ActionMessage("error.login.invalid"));
                    saveErrors(req, errors);
                    return mapping.findForward("failure");
                }
            }
        }
        if ("Clear".equals(action)) {
            req.setAttribute("errorMessageDiv", "");
            loginForm.setUserID("");
            loginForm.setPassWord("");
            return mapping.findForward("failure");
        }
        return mapping.findForward("failure");
    }
}

```
```
loginDto.java
Tệp này có thể không cần thay đổi, vì nó chỉ là DTO. Dưới đây là mã nguồn cho bạn:
package fjs.cs.dto;

public class loginDto {
    private String userID; 
    private String passWord;

    public loginDto(String userID, String passWord) {
        super();
        this.userID = userID;
        this.passWord = passWord;
    }

    public String getUserID() {
        return userID;
    }

    public void setUserID(String userID) {
        this.userID = userID;
    }

    public String getPassWord() {
        return passWord;
    }

    public void setPassWord(String passWord) {
        this.passWord = passWord;
    }
}

```
```
loginForm.java
Tệp này cũng không cần thay đổi nhiều, nhưng đây là mã cho bạn tham khảo:
package fjs.cs.form;

import org.apache.struts.action.ActionForm;

public class loginForm extends ActionForm {
    private static final long serialVersionUID = 1L;
    private String userID; 
    private String passWord;

    public loginForm() {
        super();
    }

    public String getUserID() {
        return userID;
    }

    public void setUserID(String userID) {
        this.userID = userID;
    }

    public String getPassWord() {
        return passWord;
    }

    public void setPassWord(String passWord) {
        this.passWord = passWord;
    }
}

```
```
struts-config.xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE struts-config PUBLIC
    "-//Apache Software Foundation//DTD Struts Configuration 1.3//EN"
    "http://struts.apache.org/dtds/struts-config_1_3.dtd">

<struts-config>
    <form-beans>
        <form-bean name="loginForm" type="fjs.cs.form.loginForm"/>
    </form-beans>

    <action-mappings>
        <action path="/login"
                type="fjs.cs.action.loginAction"
                name="loginForm"
                scope="request"
                input="/login.jsp">
            <forward name="success" path="/welcome.jsp"/>
            <forward name="failure" path="/login.jsp"/>
        </action>
    </action-mappings>
</struts-config>

```
```
web.xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE web-app PUBLIC 
    "-//Sun Microsystems, Inc.//DTD Web Application 2.3//EN" 
    "http://java.sun.com/dtd/web-app_2_3.dtd">

<web-app>
    <display-name>HelloStruts1x</display-name>

    <!-- Spring Context Configuration -->
    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>/WEB-INF/applicationContext.xml</param-value>
    </context-param>
    
    <!-- Spring Context Loader Listener -->
    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>

    <!-- Struts Action Servlet -->
    <servlet>
        <servlet-name>action</servlet-name>
        <servlet-class>org.apache.struts.action.ActionServlet</servlet-class>
        <init-param>
            <param-name>config</param-name>
            <param-value>/WEB-INF/struts-config.xml</param-value>
        </init-param>
        <load-on-startup>2</load-on-startup>
    </servlet>

    <servlet-mapping>
        <servlet-name>action</servlet-name>
        <url-pattern>*.do</url-pattern>
    </servlet-mapping>
    
    <welcome-file-list>
        <welcome-file>index.jsp</welcome-file>
    </welcome-file-list>
</web-app>

```
```
MstUser.hbm.xml
<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE hibernate-mapping PUBLIC
        "-//Hibernate/Hibernate Mapping DTD 3.0//EN"
        "http://hibernate.sourceforge.net/hibernate-mapping-3.0.dtd">
<hibernate-mapping>
    <class name="fjs.cs.model.MstUser" table="MSTUSERS">
        <id name="id" column="ID">
            <generator class="native"/>
        </id>
        <property name="userID" column="USERID"/>
        <property name="password" column="PASSWORD"/>
        <property name="deleteYmd" column="DELETE_YMD"/>
    </class>
</hibernate-mapping>

```
```
package fjs.cs.util;

import org.hibernate.SessionFactory;
import org.hibernate.cfg.Configuration;

public class HibernateUtil {
    private static final SessionFactory sessionFactory = buildSessionFactory();

    private static SessionFactory buildSessionFactory() {
        try {
            // Tạo một phiên làm việc từ cấu hình hibernate.cfg.xml
            return new Configuration().configure().buildSessionFactory();
        } catch (Throwable ex) {
            // Nếu không thể tạo phiên làm việc, ném ra ngoại lệ
            System.err.println("Initial SessionFactory creation failed." + ex);
            throw new ExceptionInInitializerError(ex);
        }
    }

    public static SessionFactory getSessionFactory() {
        return sessionFactory;
    }
}

```
```
Struts_Spring_Hibernate/
├── src/
│   ├── fjs/
│   │   ├── cs/
│   │   │   ├── action/
│   │   │   │   ├── loginAction.java
│   │   │   ├── dao/
│   │   │   │   ├── loginDao.java
│   │   │   ├── dto/
│   │   │   │   ├── loginDto.java
│   │   │   ├── form/
│   │   │   │   ├── loginForm.java
│   │   │   ├── util/
│   │   │   │   ├── HibernateUtil.java
│   │   │   ├── db/
│   │   │   │   ├── connectDB.java (bỏ nếu không cần)
│   │   │   ├── hbm/
│   │   │   │   ├── MstUser.hbm.xml
│   ├── resources/
│   │   ├── hibernate.cfg.xml
│   │   ├── applicationContext.xml (nếu bạn dùng Spring)
├── WEB-INF/
│   ├── struts-config.xml
│   ├── web.xml
│   ├── login.jsp
│   ├── welcome.jsp

```






```
package fjs.cs.action;

import java.sql.SQLException;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import org.apache.struts.action.Action;
import org.apache.struts.action.ActionForm;
import org.apache.struts.action.ActionForward;
import org.apache.struts.action.ActionMapping;

import fjs.cs.form.loginForm;
import fjs.cs.handler.LoginHandler;

public class loginAction extends Action {

    private LoginHandler loginHandler = new LoginHandler(); // Khởi tạo handler ở đây

    public ActionForward execute(ActionMapping mapping, ActionForm form,
                                 HttpServletRequest req, HttpServletResponse resp) throws SQLException {

        loginForm loginForm = (loginForm) form;
        String action = req.getParameter("action");

        if ("Login".equals(action)) {
            return loginHandler.handleLogin(mapping, req, loginForm);
        } 
        if ("Clear".equals(action)) {
            return loginHandler.handleClear(mapping, req, loginForm);
        }

        return mapping.findForward("failure");
    }
}

```
```
package fjs.cs.handler;

import java.sql.SQLException;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpSession;

import org.apache.struts.action.ActionForward;
import org.apache.struts.action.ActionMapping;
import org.apache.struts.action.ActionMessage;
import org.apache.struts.action.ActionMessages;

import fjs.cs.dao.loginDao;
import fjs.cs.form.loginForm;

public class LoginHandler {
    
    private loginDao loginDao;

    public LoginHandler() {
        this.loginDao = new loginDao();
    }

    // Hàm xử lý đăng nhập
    public ActionForward handleLogin(ActionMapping mapping, HttpServletRequest req, loginForm loginForm) throws SQLException {
        String userid = loginForm.getUserID();
        String password = loginForm.getPassWord();
        ActionMessages errors = new ActionMessages();

        // Kiểm tra dữ liệu đầu vào
        if (isNullOrEmpty(userid)) {
            errors.add("username", new ActionMessage("error.username.required"));
        }
        if (isNullOrEmpty(password)) {
            errors.add("password", new ActionMessage("error.password.required"));
        }
        if (!errors.isEmpty()) {
            saveErrors(req, errors);
            return mapping.findForward("failure");
        }

        // Kiểm tra đăng nhập
        int cnt = loginDao.checkLogin(userid, password);
        if (cnt == 1) {
            HttpSession session = req.getSession();
            session.setAttribute("userid", userid);
            return mapping.findForward("success");
        } else {
            errors.add("login", new ActionMessage("error.login.invalid"));
            saveErrors(req, errors);
            return mapping.findForward("failure");
        }
    }

    // Hàm xử lý clear
    public ActionForward handleClear(ActionMapping mapping, HttpServletRequest req, loginForm loginForm) {
        req.setAttribute("errorMessageDiv", ""); // Reset thông báo lỗi
        loginForm.setUserID("");
        loginForm.setPassWord("");
        return mapping.findForward("failure");
    }

    // Hàm kiểm tra chuỗi rỗng hoặc null
    private boolean isNullOrEmpty(String str) {
        return str == null || str.trim().isEmpty();
    }
}

```
```
constants
package fjs.cs.util;

public class ActionConstants {
    public static final String ACTION_LOGIN = "Login";
    public static final String ACTION_CLEAR = "Clear";

    public static final String SESSION_USERID = "userid";

    public static final String ERROR_USERNAME_REQUIRED = "error.username.required";
    public static final String ERROR_PASSWORD_REQUIRED = "error.password.required";
    public static final String ERROR_LOGIN_INVALID = "error.login.invalid";

    public static final String FORWARD_SUCCESS = "success";
    public static final String FORWARD_FAILURE = "failure";
}

```
-----------------------------------------DAY 3 ----------------------------------------------------
```
xóa @id @entity
Thêm cấu hình cho Hibernate để sử dụng file ánh xạ XML:
<bean id="sessionFactory" class="org.springframework.orm.hibernate5.LocalSessionFactoryBean">
    <property name="dataSource" ref="dataSource"/>
    <property name="mappingResources">
        <list>
            <value>User.hbm.xml</value>
        </list>
    </property>
    <property name="hibernateProperties">
        <props>
            <prop key="hibernate.dialect">org.hibernate.dialect.SQLServerDialect</prop>
            <prop key="hibernate.show_sql">true</prop>
        </props>
    </property>
</bean>

```
```
Tạo file User.hbm.xml trong thư mục src/main/resources hoặc WEB-INF:
<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE hibernate-mapping PUBLIC
        "-//Hibernate/Hibernate Mapping DTD 3.0//EN"
        "http://www.hibernate.org/dtd/hibernate-mapping-3.0.dtd">
<hibernate-mapping>
    <class name="com.example.model.User" table="Users">
        <id name="username" column="username">
            <generator class="assigned"/>
        </id>
        <property name="password" column="password"/>
    </class>
</hibernate-mapping>
```
-----------------------------------------DAY 2 ----------------------------------------------------
```
web.xml
<web-app>
    <!-- Spring ContextLoaderListener -->
    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>

    <!-- Struts ActionServlet -->
    <servlet>
        <servlet-name>action</servlet-name>
        <servlet-class>org.apache.struts.action.ActionServlet</servlet-class>
        <init-param>
            <param-name>config</param-name>
            <param-value>/WEB-INF/struts-config.xml</param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
    </servlet>

    <servlet-mapping>
        <servlet-name>action</servlet-name>
        <url-pattern>*.do</url-pattern>
    </servlet-mapping>
</web-app>
```
```
struts-config.xml trong thư mục WEB-INF:
<struts-config>
    <form-beans>
        <form-bean name="loginForm" type="com.example.form.LoginForm"/>
    </form-beans>

    <action-mappings>
        <action path="/login" type="com.example.action.LoginAction" 
                name="loginForm" scope="request" input="/jsp/login.jsp">
            <forward name="success" path="/jsp/welcome.jsp"/>
            <forward name="failure" path="/jsp/login.jsp"/>
        </action>
    </action-mappings>
</struts-config>
```
```
Tạo lớp LoginForm trong package com.example.form:
package com.example.form;

import org.apache.struts.action.ActionForm;

public class LoginForm extends ActionForm {
    private String username;
    private String password;

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }
}
```
```
Tạo lớp LoginAction trong package com.example.action:
package com.example.action;

import com.example.service.UserService;
import com.example.form.LoginForm;
import org.apache.struts.action.ActionForm;
import org.apache.struts.action.ActionForward;
import org.apache.struts.action.ActionMapping;
import org.apache.struts.action.Action;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

public class LoginAction extends Action {

    private UserService userService;

    public void setUserService(UserService userService) {
        this.userService = userService;
    }

    @Override
    public ActionForward execute(ActionMapping mapping, ActionForm form,
                                 HttpServletRequest request, HttpServletResponse response) {
        LoginForm loginForm = (LoginForm) form;

        String username = loginForm.getUsername();
        String password = loginForm.getPassword();

        if (userService.isValidUser(username, password)) {
            return mapping.findForward("success");
        } else {
            request.setAttribute("errorMessage", "Invalid username or password");
            return mapping.findForward("failure");
        }
    }
}
```
```
Tạo lớp UserService
Tạo lớp UserService trong package com.example.service:
package com.example.service;

import com.example.dao.UserDAO;
import com.example.model.User;

public class UserService {

    private UserDAO userDAO;

    public void setUserDAO(UserDAO userDAO) {
        this.userDAO = userDAO;
    }

    public boolean isValidUser(String username, String password) {
        User user = userDAO.findByUsername(username);
        return user != null && user.getPassword().equals(password);
    }
}
```
```
Tạo lớp UserDAO trong package com.example.dao:
package com.example.dao;

import com.example.model.User;
import org.hibernate.SessionFactory;
import org.hibernate.Session;

public class UserDAO {

    private SessionFactory sessionFactory;

    public void setSessionFactory(SessionFactory sessionFactory) {
        this.sessionFactory = sessionFactory;
    }

    public User findByUsername(String username) {
        Session session = sessionFactory.getCurrentSession();
        return session.createQuery("FROM User WHERE username = :username", User.class)
                      .setParameter("username", username)
                      .uniqueResult();
    }
}
```
```
Tạo lớp User trong package com.example.model:
package com.example.model;

import javax.persistence.Entity;
import javax.persistence.Id;

@Entity
public class User {

    @Id
    private String username;
    private String password;

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }
}
```
```
Tạo file applicationContext.xml trong thư mục WEB-INF:
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans 
                           http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
        <property name="driverClassName" value="com.mysql.cj.jdbc.Driver"/>
        <property name="url" value="jdbc:mysql://localhost:3306/your_database"/>
        <property name="username" value="your_username"/>
        <property name="password" value="your_password"/>
    </bean>

    <bean id="sessionFactory" class="org.springframework.orm.hibernate5.LocalSessionFactoryBean">
        <property name="dataSource" ref="dataSource"/>
        <property name="packagesToScan" value="com.example.model"/>
        <property name="hibernateProperties">
            <props>
                <prop key="hibernate.dialect">org.hibernate.dialect.MySQLDialect</prop>
                <prop key="hibernate.show_sql">true</prop>
            </props>
        </property>
    </bean>

    <bean id="transactionManager" class="org.springframework.orm.hibernate5.HibernateTransactionManager">
        <property name="sessionFactory" ref="sessionFactory"/>
    </bean>

    <!-- Cấu hình UserDAO và UserService -->
    <bean id="userDAO" class="com.example.dao.UserDAO">
        <property name="sessionFactory" ref="sessionFactory"/>
    </bean>

    <bean id="userService" class="com.example.service.UserService">
        <property name="userDAO" ref="userDAO"/>
    </bean>

    <!-- Cấu hình LoginAction -->
    <bean id="loginAction" class="com.example.action.LoginAction">
        <property name="userService" ref="userService"/>
    </bean>
</beans>
```
```
Tạo file login.jsp trong thư mục jsp:
<%@ taglib uri="http://struts.apache.org/tags-html" prefix="html" %>
<html>
<head>
    <title>Login</title>
</head>
<body>
    <h2>Login Form</h2>
    <html:form action="/login">
        <html:text property="username" placeholder="Username"/>
        <html:text property="password" placeholder="Password" type="password"/>
        <html:submit value="Login"/>
    </html:form>
    <c:if test="${not empty errorMessage}">
        <p style="color:red;">${errorMessage}</p>
    </c:if>
</body>
</html>
```
```
Tạo file welcome.jsp trong thư mục jsp:
<html>
<head>
    <title>Welcome</title>
</head>
<body>
    <h2>Welcome to the application!</h2>
</body>
</html>
```
```
Thay đổi phần cấu hình dataSource trong file applicationContext.xml để phù hợp với SQL Server:
<bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
    <property name="driverClassName" value="com.microsoft.sqlserver.jdbc.SQLServerDriver"/>
    <property name="url" value="jdbc:sqlserver://localhost:1433;databaseName=your_database"/>
    <property name="username" value="your_username"/>
    <property name="password" value="your_password"/>
</bean>
```
```
Cũng cần thay đổi cấu hình cho Hibernate để sử dụng SQL Server:
<bean id="sessionFactory" class="org.springframework.orm.hibernate5.LocalSessionFactoryBean">
    <property name="dataSource" ref="dataSource"/>
    <property name="packagesToScan" value="com.example.model"/>
    <property name="hibernateProperties">
        <props>
            <prop key="hibernate.dialect">org.hibernate.dialect.SQLServerDialect</prop>
            <prop key="hibernate.show_sql">true</prop>
        </props>
    </property>
</bean>
http://localhost:8080/your_project_name/jsp/login.jsp.
```
