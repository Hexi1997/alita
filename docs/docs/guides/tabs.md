# 多 Tabs （实验性）

## 开始使用

配置 `tabsLayout` 开启 

```ts
// config/config.ts
export default {
  tabsLayout: { },
};
```

## 配置

### 构建时配置

> config/config.ts 中配置 tabsLayout

```ts
import { defineConfig } from 'umi';
 
export default defineConfig({
  layout: {
    hasCustomTabs: Joi.boolean(),
    hasDropdown: Joi.boolean(),
    hasFixedHeader: Joi.boolean()
  },
});
```

#### hasCustomTabs

是否使用自定义的 tabs 组件，需要搭配运行时配置 `getCustomTabs` 使用(详见下方运行时配置说明)

#### hasDropdown 

是否开启右侧的 tabs 管理器，可以实现“关闭左侧”，“关闭右侧”，“关闭其他”和“刷新”等功能。

#### hasFixedHeader

是否开启 tabs 头部定位

### 运行时配置

> src/app.ts 中配置

```tsx

import { Tabs, message} from 'antd';
import { EllipsisOutlined } from "@ant-design/icons";

const { TabPane } = Tabs;

export function tabsLayout({initialState}) {
  // 如果和 initialState 插件搭配使用的时候，会传入 initialState，否则 initialState 永远为空
  return {
    local: {
      '/': '首页',
      '/users': '用户',
      '/foo': '其他',
    },
    icon: {
      '/users': <UserOutlined />,
    },
    // 其他 antd Tabs 属性
    size: 'large',
    onTabClick: (key: string) => {
      console.log(key);
    },
  };
}

export const getCustomTabs = () => {
  return ({
    isKeep,
    keepElements,
    navigate,
    dropByCacheKey,
    local,
    activeKey,
  }: any) => {
    return (
      <div className="rumtime-keep-alive-tabs-layout" hidden={!isKeep}>
        <Tabs
          hideAdd
          onChange={(key: string) => {
            navigate(key);
          }}
          activeKey={activeKey}
          type="editable-card"
          onEdit={(targetKey: string) => {
            let newActiveKey = activeKey;
            let lastIndex = -1;
            const newPanel = Object.keys(keepElements.current);
            for (let i = 0; i < newPanel.length; i++) {
              if (newPanel[i] === targetKey) {
                lastIndex = i - 1;
              }
            }
            const newPanes = newPanel.filter((pane) => pane !== targetKey);
            if (newPanes.length && newActiveKey === targetKey) {
              if (lastIndex >= 0) {
                newActiveKey = newPanes[lastIndex];
              } else {
                newActiveKey = newPanes[0];
              }
            }
            if (lastIndex === -1 && targetKey === location.pathname) {
              message.info('至少要保留一个窗口');
            } else {
              dropByCacheKey(targetKey);
              if (newActiveKey !== location.pathname) {
                navigate(newActiveKey);
              }
            }
          }}
        >
          {Object.entries(keepElements.current).map(
            ([pathname, element]: any) => (
              <TabPane tab={`${local[pathname] || pathname}`} key={pathname} />
            ),
          )}
        </Tabs>
      </div>
    );
  };
};

```

#### tabsLayout

> 如果和 initialState 插件搭配使用的时候，tabsLayout 函数会传入真实的 initialState，否则 initialState 永远为空

tabsLayout 返回一个对象，除了 icon 和 local 为多Tabs相关功能使用，剩余的属性会透传到 Tabs 上，你可以在这里设置 Tabs。

#### getCustomTabs

完全覆盖内置的多 Tabs 组件，需要搭配配置 `hasCustomTabs:true` 使用。

## API

### KeepAliveContext

可以从 alita 中找到 KeepAliveContext 上下文。


```ts
import { KeepAliveContext } from 'alita';

export default () => {
  const {  } = React.useContext(KeepAliveContext);
  return (
  );
};
```
#### dropByCacheKey

清空当前页面的状态保持，其实也就是刷新页面。

#### updateTab

更新当前 tab 的属性。可以用于透传参数到 TabPane 组件上。

```ts

import { TabPaneProps } from 'antd';

export interface TabConfig extends TabPaneProps{
  icon?: ReactNode;
  name?: string;
  closable?: boolean;
}

const updateTab = (path: string, config: TabConfig) => void,
```

使用方法

```ts
import { KeepAliveContext, useLocation, useParams } from 'alita';

export default () => {
  const location = useLocation();
  const { updateTab } = React.useContext(KeepAliveContext);
  const params = useParams();

  useEffect(() => {
    updateTab(location.pathname, {
      name: '详情' + params?.index,
      closeIcon: <UserOutlined/>,
    });
  }, []);

  const handleClick = () => {
    updateTab(location.pathname, {
      icon: <UserOutlined />,
      name: 'hahaha' + Math.ceil((Math.random() * 100) / 10),
      closable: false,
    });
  };

  return (
    
  );
};

```

####  其他不常用

```ts
  // 关闭左侧 tabs
  dropLeftTabs: (path: string) => void,
  // 关闭右侧 tabs
  dropRightTabs: (path: string) => void,
  // 关闭其他
  dropOtherTabs: (path: string) => void,
  // 刷新
  refreshTab: (path: string) => void,
```