<template>
  <div style="display: block;flex: 1;height: 100%" class="w-custom-menu">
    <w-row>
      <w-col :span="24">
        <w-custom-menu-header></w-custom-menu-header>
      </w-col>
    </w-row>
    <w-row>
      <w-col :span="24">
        <div class="container-box">
          <!--      左侧菜单编辑器-->
          <div class="phone-box">
            <div class="menu__preview-hd" />
            <div class="menu__preview-bd">
              <div class="menu__preview-bottom">
                <div class="menu__keyboard">
                  <div class="menu__keyboard-icon" />
                </div>
                <div class="menu-list">
                  <w-button v-if="menu_data.button.length <= 0" type="text" class="menu-add-no-menu"
                            @click="addMainMenu">
                    <div class="add-no-menu">
                      <div class="menu__add-icon" />
                      <span>添加菜单</span>
                    </div>
                  </w-button>
                  <div
                    v-for="item in menu_data.button"
                    v-else
                    :key="item.id"
                    :class="`menu-list-item ${select_menu_id === item.id && select_menu_sub_id === 0?'menu-box-shadow':''}`"
                  >
                    <div class="menu__preview-line" />
                    <div
                      :class="`menu-item ${select_menu_id === item.id && select_menu_sub_id === 0?'menu-item-color':''}`"
                      @click="selectMenu(item.id)"
                    >{{ item.name }}
                    </div>
                    <!--                  二级菜单-->
                    <div v-if="select_menu_id === item.id" class="submenu">
                      <!--                    二级-->
                      <div
                        v-for="sub_item in item.sub_button"
                        :key="sub_item.id"
                        :class="`menu-item-sub ${select_menu_sub_id === sub_item.id?'menu-box-shadow menu-item-color':''} ${select_menu_sub_id !== sub_item.id?'menu-item-border-color':''}`"
                      >
                        <span
                          style="flex: 1;height: 100%;display: flex;justify-content: center;align-items: center"
                          @click="selectSubMenu(sub_item.id)"
                        >{{ sub_item.name }}</span>
                        <div class="sub-menu-bar">
                          <div v-if="select_menu_sub_id === sub_item.id" class="sub-menu-bar-box">
                            <div class="sub-center-bar bar-hover" @click="onDeleteSub" />
                          </div>
                        </div>
                      </div>
                      <!--                    二级增加按钮-->
                      <div v-if="item.sub_button.length < 5" class="add-button-sub">
                        <w-button type="text" title="最多添加5个子菜单" @click="addMSubMenu">
                          <template #icon>
                            <icon-plus />
                          </template>
                          <!-- Use the default slot to avoid extra spaces -->
                          <template #default>添加</template>
                        </w-button>
                      </div>
                      <i
                        :class="`arrow arrow_out ${select_menu_sub_id === 5 && item.sub_button.length === 5?'arrow_out_select':''}`"
                      />
                      <i
                        :class="`arrow arrow_in ${select_menu_sub_id === 5 && item.sub_button.length === 5?'arrow_in_select':''}`"
                      />
                    </div>
                    <div class="menu-bar">
                      <div
                        v-if="select_menu_id === item.id"
                        :class="`menu-bar-box ${menu_data.button.length > 1?'bar-padding':''}`"
                      >
                        <div v-if="item.id !== menu_data.button[0].id" class="left-bar bar-hover" @click="onLeftMove" />
                        <div class="center-bar bar-hover" @click="onDelete" />
                        <div
                          v-if="menu_data.button[1] && item.id !== menu_data.button[1].id && menu_data.button.length === 2"
                          class="right-bar bar-hover"
                          @click="onRightMove"
                        />
                        <div
                          v-if="menu_data.button.length > 2 && item.id !== menu_data.button[2].id"
                          class="right-bar bar-hover"
                          @click="onRightMove"
                        />
                      </div>
                    </div>
                  </div>
                  <!--                  主菜单添加按钮-->
                  <div v-if="menu_data.button.length < 3 && menu_data.button.length > 0" class="add-button">
                    <div class="menu_button_preview-line" />
                    <w-button type="text" @click="addMainMenu">
                      <template #icon>
                        <icon-plus />
                      </template>
                    </w-button>
                  </div>
                </div>
              </div>
            </div>
          </div>
          <!--        右侧编辑器-->
          <div class="right-editor-box">
            <div v-if="menu_data.button.length > 0" class="attribute-box">
              <!--              没有有二级菜单-->
              <div v-if="have_sub_menu === false && select_menu_id !== 0 || select_menu_sub_id !== 0"
                   class="custom-no-have-menu-box">
                <h3 style="padding-bottom: 25px">菜单信息</h3>
                <w-form ref="formVisible" :model="formData" :rules="rule" :style="{ width: '500px' }">
                  <div class="custom-menu-name">
                    <w-form-item label="名称" field="name" required>
                      <w-input v-model="formData.name" placeholder="仅支持中文或数字" />
                      <template #extra>
                        <div>仅支持中英文和数字，字数不超过4个汉字</div>
                      </template>
                    </w-form-item>
                  </div>
                  <div class="custom-menu-type">
                    <w-form-item label="消息类型">
                      <w-radio-group v-model="menu_type" :default-value="menu_type" :options="menu_type_options"
                                     class="ml-4" />
                    </w-form-item>
                  </div>
                  <div v-if="menu_type === 'click'" class="custom-menu-content">
                    <w-form-item label="菜单内容" field="key" required>
                      <w-input v-model="formData.key" placeholder="仅支持中文或数字" />
                      <template #extra>
                        <div>key值为管理平台创建好的功能key</div>
                      </template>
                    </w-form-item>
                  </div>
                  <div v-if="menu_type === 'view'" class="custom-menu-content">
                    <w-form-item label="网页链接" field="url" required>
                      <w-input v-model="formData.url" placeholder="公众号链接" />
                      <template #extra>
                        <div>跳转连接推荐使用安全域名https://</div>
                      </template>
                    </w-form-item>
                  </div>
                  <div v-if="menu_type === 'miniprogram'" class="custom-menu-content">
                    <w-form-item label="Appid" field="appid" required>
                      <w-input v-model="formData.appid" placeholder="小程序Appid" />
                      <template #extra>
                        <div>输入对应的小程序Appid 示例：wxd027d2b162044fd5</div>
                      </template>
                    </w-form-item>
                    <w-form-item label="页面路径" field="pagepath" required>
                      <w-input v-model="formData.pagepath" placeholder="页面路径" />
                      <template #extra>
                        <div>输入对应的小程序页面路径 示例：/pages/index/index</div>
                      </template>
                    </w-form-item>
                  </div>
                </w-form>
                <div class="custom-menu-delete">
                  <div class="input-title">
                    <w-popconfirm content="删除确认" type="error" @ok="onDelete">
                      <w-button type="text" class="delete-btn" style="color: #f1164e">删除菜单</w-button>
                    </w-popconfirm>
                  </div>
                </div>
              </div>
              <!--              有二级菜单-->
              <div v-if="have_sub_menu === true && select_menu_id !== 0 && select_menu_sub_id === 0"
                   class="custom-have-menu-box">
                <h3 style="padding-bottom: 25px;">菜单信息</h3>
                <w-form ref="formVisible" :model="formData" :rules="rule">
                  <div class="custom-menu-name">
                    <w-form-item label="名称" field="name" required>
                      <w-input v-model="formData.name" placeholder="仅支持中文或数字" />
                      <template #extra>
                        <div>仅支持中英文和数字，字数不超过4个汉字</div>
                      </template>
                    </w-form-item>
                  </div>
                  <div class="custom-menu-delete">
                    <div class="input-title">
                      <w-popconfirm content="删除确认" type="error" @ok="onDelete">
                        <w-button type="text" class="delete-btn" style="color: #f1164e">删除菜单</w-button>
                      </w-popconfirm>
                    </div>
                  </div>
                </w-form>
              </div>
            </div>
            <!--            没有菜单-->
            <div v-if="menu_data.button.length === 0" class="attribute-box">
              <w-empty description="你未添加自定义菜单，点击左侧添加菜单为公众号创建菜单栏。" />
            </div>
            <!--            右侧底部按钮组-->
            <div class="menu__edit-action-bar">
              <div class="menu__edit-action-inner">
                <div class="menu__edit-button">
                  <w-button @click="preview">预览</w-button>
                  <w-button type="primary" @click="submitMenuData">保存并发布</w-button>
                </div>
              </div>
            </div>
          </div>
        </div>
      </w-col>
    </w-row>
  </div>
</template>

<script setup lang="ts" name="custom-menu">
import {ref,watch,PropType,onMounted} from 'vue'
import {Modal, Notification,Message} from '@arco-design/web-vue';
import WCustomMenuHeader from './components/header.vue'

const select_menu_id = ref(0)
const select_menu_sub_id = ref(0)
const have_sub_menu = ref(false)

// 菜单类型
const menu_type = ref('')
const menu_type_options = ref([
  {label: '推送事件消息', value: 'click'},
  {label: '网页跳转', value: 'view'},
  {label: '小程序跳转', value: 'miniprogram'}
])
// 表单数据
const formData = ref({
  name: '',
  key: '',
  url: '',
  appid: '',
  pagepath: ''
} as any)

// 验证规则
const rule = ref({
  name: [
    {required: true, message: '请输入菜单名称', trigger: 'blur'},
    {minLength: 2, message: '请输入菜单名称 最低5位字符', trigger: 'blur'},
    {maxLength: 4, message: '请输入菜单名称 最多4位字符', trigger: 'blur'}
  ],
  url: [
    {required: true, message: '请输入网页链接', trigger: 'blur'},
    {match: /^(http|https):\/\//, message: '请输入正确的网页链接', trigger: 'blur'}
  ],
  appid: [
    {required: true, message: '请输入小程序Appid', trigger: 'blur'},
    {match: /^[w-zA-Z0-9]{18}$/, message: '请输入正确的小程序Appid', trigger: 'blur'}
  ],
  pagepath: [
    {required: true, message: '请输入页面路径', trigger: 'blur'},
    // 是否是小程序页面路径/pages/index/index
    {match: /^\/pages\/[w-zA-Z0-9]+\/[w-zA-Z0-9]+$/, message: '请输入正确的页面路径', trigger: 'blur'}
  ],
  key: [
    {required: true, message: '请输入菜单内容'}
  ]
})



// 组件props
const props = defineProps({
  isRemote: {
    type: Boolean,
    default: false
  },
  menuData: {
    type: Object as PropType<Record<any, any>>,
    default: () => {
      return {
        button: ([] as Record<any, any>)
      }
    }
  }
})

// 菜单数据
let menu_data = ref({
  button: [] as Record<any, any>,
})

watch(props, (newVal) => {
  if (newVal.menuData) {
    initData()
  }
})

// 选择菜单
const selectMenu = async (item: number) => {
  if (item === 0) {
    return
  }
  formData.value = {}
  select_menu_id.value = item
  select_menu_sub_id.value = 0
  const menu = menu_data.value.button.find((item: any) => item.id === select_menu_id.value)
  // 如果没有二级菜单，将form表单的数据设置为一级菜单的数据
  if (menu_data.value.button.find((item: any) => item.id === select_menu_id.value)?.sub_button.length === 0) {
    have_sub_menu.value = false
    formData.value.name = menu.name
    menu_type.value = menu.type
    switch (menu_type.value) {
      case 'click':
        formData.value.key = menu.key
        break
      case 'view':
        formData.value.url = menu.url
        break
      case 'miniprogram':
        formData.value.appid = menu.appid
        formData.value.pagepath = menu.pagepath
        break
    }
  } else {
    have_sub_menu.value = true
    formData.value.name = menu?.name
  }
}

// 选择二级菜单
const selectSubMenu = async (item: number) => {
  if (item === 0) {
    return
  }
  formData.value = {}
  select_menu_sub_id.value = item
  const menu = menu_data.value.button.find((item: any) => item.id === select_menu_id.value)
  const sub_menu = menu.sub_button.find((item: any) => item.id === select_menu_sub_id.value)
  formData.value.name = sub_menu.name
  switch (sub_menu.type) {
    case 'click':
      menu_type.value = 'click'
      formData.value.key = sub_menu.key
      break
    case 'view':
      menu_type.value = 'view'
      formData.value.url = sub_menu.url
      break
    case 'miniprogram':
      menu_type.value = 'miniprogram'
      formData.value.appid = sub_menu.appid
      formData.value.pagepath = sub_menu.pagepath
      break
  }
}


// 初始化数据
const initData = async () => {
  if (props.isRemote) {
    // 如果是远程数据 将远程数据赋值给menu_data 给每个菜单添加id
    props.menuData.button.forEach((item: any, index: number) => {
      item.id = index + 1
      if (item.sub_button.length > 0) {
        item.sub_button.forEach((sub_item: any, sub_index: number) => {
          sub_item.id = sub_index + 1
        })
      }
    })
    menu_data.value.button = props.menuData.button
  }
  if (menu_data.value.button.length > 0) {
    select_menu_id.value = menu_data.value.button[0].id
    await selectMenu(select_menu_id.value)
  }
}

// 保存菜单数据
const saveMenu = async () => {
  // 如果没有选择菜单，不保存
  if (select_menu_id.value === 0) {
    return
  }
  // 找到对应的菜单
  const menu = menu_data.value.button.find((item: any) => item.id === select_menu_id.value)
  // 如果没有选择二级菜单
  if (select_menu_sub_id.value === 0) {
    // 如果没有二级菜单，修改完整的一级菜单 如果有二级菜单，修改一级菜单的name
    if (menu.sub_button.length === 0) {
      menu.name = formData.value.name
      menu.type = menu_type.value
      switch (menu_type.value) {
        case 'click':
          menu.key = formData.value.key
          // 把当前一级菜单除了name、sub_button、id、type、key字段外 删除一级菜单的其他字段
          const {name, sub_button, id, type, key} = menu
          menu_data.value.button.splice(
            menu_data.value.button.findIndex((item: any) => item.id === select_menu_id.value),
            1,
            {
              name,
              sub_button,
              id,
              type,
              key
            }
          )
          break
        case 'view':
          menu.url = formData.value.url
          // 把当前一级菜单除了name、sub_button、id、type、url字段外 删除一级菜单的其他字段
          const {name: name1, sub_button: sub_button1, id: id1, type: type1, url} = menu
          menu_data.value.button.splice(
            menu_data.value.button.findIndex((item: any) => item.id === select_menu_id.value),
            1,
            {
              name: name1,
              sub_button: sub_button1,
              id: id1,
              type: type1,
              url
            }
          )
          break
        case 'miniprogram':
          menu.appid = formData.value.appid
          menu.pagepath = formData.value.pagepath
          // 把当前一级菜单除了name、sub_button、id、type、appid、pagepath字段外 删除一级菜单的其他字段
          const {name: name2, sub_button: sub_button2, id: id2, type: type2, appid, pagepath} = menu
          menu_data.value.button.splice(
            menu_data.value.button.findIndex((item: any) => item.id === select_menu_id.value),
            1,
            {
              name: name2,
              sub_button: sub_button2,
              id: id2,
              type: type2,
              appid,
              pagepath
            }
          )
          break
      }
    } else {
      menu.name = formData.value.name
    }
  } else {
    const sub_menu = menu.sub_button.find((item: any) => item.id === select_menu_sub_id.value)
    sub_menu.name = formData.value.name
    sub_menu.type = menu_type.value
    switch (menu_type.value) {
      case 'click':
        sub_menu.key = formData.value.key
        // 把当前二级菜单除了name、id、type、key字段外 删除一级菜单的其他字段
        const {name, id, type, key} = sub_menu
        menu.sub_button.splice(
          menu.sub_button.findIndex((item: any) => item.id === select_menu_sub_id.value),
          1,
          {
            name,
            id,
            type,
            key
          }
        )
        break
      case 'view':
        sub_menu.url = formData.value.url
        // 把当前二级菜单除了name、id、type、url字段外 删除一级菜单的其他字段
        const {name: name1, id: id1, type: type1, url} = sub_menu
        menu.sub_button.splice(
          menu.sub_button.findIndex((item: any) => item.id === select_menu_sub_id.value),
          1,
          {
            name: name1,
            id: id1,
            type: type1,
            url
          }
        )
        break
      case 'miniprogram':
        sub_menu.appid = formData.value.appid
        sub_menu.pagepath = formData.value.pagepath
        // 把当前二级菜单除了name、id、type、appid、pagepath字段外 删除一级菜单's其他字段
        const {name: name2, id: id2, type: type2, appid, pagepath} = sub_menu
        menu.sub_button.splice(
          menu.sub_button.findIndex((item: any) => item.id === select_menu_sub_id.value),
          1,
          {
            name: name2,
            id: id2,
            type: type2,
            appid,
            pagepath
          }
        )
        break
    }
    // 消息提示保存成功
    Message.success('保存成功')
  }
}

// 提交菜单数据
const emits = defineEmits(['submitData'])

// 提交菜单数据
const submitMenuData = async () => {
  if (menu_data.value.button.length === 0) {
    Message.error('请添加菜单')
    return
  }
  await saveMenu()
  emits('submitData', menu_data.value)
}

// 预览
const preview = () => {
  Message.warning('暂不支持预览')
}

// 是否包含二级菜单
const isHaveSubMenu = async () => {
  if (menu_data.value.button.length > 0) {
    const menu_id = select_menu_id.value
    const menu = menu_data.value.button.find((item: any) => item.id === menu_id)
    // 如果存在二级菜单
    return menu.sub_button !== undefined && menu.sub_button !== null && menu.sub_button.length > 0;
  }
}

// 添加一级菜单
const addMainMenu = async () => {
  formData.value = {}
  if (menu_data.value.button.length < 3) {
    let menu_id = 0
    if (menu_data.value.button.length === 0) {
      menu_id = 1
    } else {
      menu_id = menu_data.value.button[menu_data.value.button.length - 1].id + 1
    }
    menu_data.value.button.push({
      id: menu_id,
      name: '菜单名称',
      type: 'click',
      key: '',
      sub_button: []
    })
    // 如果新增菜单将菜单id设置为新增菜单的id
    select_menu_id.value = menu_id
    await selectMenu(select_menu_id.value)
  }
}

// 添加二级菜单
const addMSubMenu = async () => {
  formData.value = {}
  const menu_id = select_menu_id.value
  const menu = menu_data.value.button.find((item: any) => item.id === menu_id)
  // 如果二级菜单小于5个
  if (menu.sub_button.length < 5) {
    let sub_menu_id = 0
    if (menu.sub_button.length === 0) {
      // 如果没有二级菜单，设置id为1
      sub_menu_id = 1
    } else {
      // 如果有二级菜单，设置id为最后一个二级菜单的id+1
      sub_menu_id = menu.sub_button[menu.sub_button.length - 1].id + 1
    }
    menu.sub_button.push({
      // 添加二级菜单
      id: menu.sub_button.length + 1,
      name: '子菜单名称',
      type: 'click',
      key: ''
    })
    select_menu_sub_id.value = sub_menu_id
    // 将选中二级菜单id设置为新增二级菜单的id
    await selectSubMenu(select_menu_sub_id.value)
    // 把当前一级菜单除了name和sub_button和id字段外 删除一级菜单的其他字段
    const {name, sub_button, id} = menu
    menu_data.value.button.splice(
      menu_data.value.button.findIndex((item: any) => item.id === menu_id),
      1,
      {
        name,
        sub_button,
        id
      }
    )
  }
}

// 往左移动选择菜单
const onLeftMove = async () => {
  let previousId = 0
  for (let i = 0; i < menu_data.value.button.length; i++) {
    if (menu_data.value.button[i].id === select_menu_id.value) {
      if (i > 0) {
        previousId = menu_data.value.button[i - 1].id
      }
      break // 找到目标后可以提前退出循环
    }
  }
  await selectMenu(previousId)
}

// 删除对应id的菜单
const onDelete = async () => {
  const menu_id = select_menu_id.value
  for (let i = 0; i < menu_data.value.button.length; i++) {
    if (menu_data.value.button[i].id === menu_id) {
      menu_data.value.button.splice(i, 1)
      // 如果菜单数量大于0 把相邻的菜单设置为选中
      if (menu_data.value.button.length > 0) {
        if (i === menu_data.value.button.length) {
          select_menu_id.value = menu_data.value.button[i - 1].id
          await selectMenu(select_menu_id.value)
        } else {
          select_menu_id.value = menu_data.value.button[i].id
          await selectMenu(select_menu_id.value)
        }
      }

      break // 可以提高性能，因为一旦找到要删除的对象就不需要再继续循环
    }
  }
}
// 删除对应id的二级菜单
const onDeleteSub = async () => {
  const menu_id = select_menu_id.value
  const sub_menu_id = select_menu_sub_id.value
  for (let i = 0; i < menu_data.value.button.length; i++) {
    if (menu_data.value.button[i].id === menu_id) {
      for (let j = 0; j < menu_data.value.button[i].sub_button.length; j++) {
        if (menu_data.value.button[i].sub_button[j].id === sub_menu_id) {
          menu_data.value.button[i].sub_button.splice(j, 1)

          // 如果删除的是最后一个二级菜单，将一级菜单的id设置为选中
          if (menu_data.value.button[i].sub_button.length === 0) {
            select_menu_id.value = menu_id
            await selectMenu(select_menu_id.value)
          } else {
            // 如果删除的不是最后一个二级菜单，将剩余最后一个 或者前一个二级菜单设置为选中
            if (j === menu_data.value.button[i].sub_button.length) {
              select_menu_sub_id.value = menu_data.value.button[i].sub_button[j - 1].id
              await selectSubMenu(select_menu_sub_id.value)
            } else {
              select_menu_sub_id.value = menu_data.value.button[i].sub_button[j].id
              await selectSubMenu(select_menu_sub_id.value)
            }
          }

          break // 可以提高性能，因为一旦找到要删除的对象就不需要再继续循环
        }
      }
    }
  }
}

// 往右移动选择菜单
const onRightMove = async () => {
  let nextId = 0

  for (let i = 0; i < menu_data.value.button.length; i++) {
    if (menu_data.value.button[i].id === select_menu_id.value) {
      if (i < menu_data.value.button.length - 1) {
        nextId = menu_data.value.button[i + 1].id
      }
      break // 找到目标后可以提前退出循环
    }
  }
  await selectMenu(nextId)
}


</script>

<style scoped lang="scss">
@import "./style/wechat-menu";
</style>
