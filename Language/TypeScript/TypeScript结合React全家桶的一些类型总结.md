### 一、 React本身常用类型

无状态组件，函数的类型定义，FunctionComponent<P={}>、简写FC<P={}>一个泛型接口，可以接受一个参数，可以不传,用来定义props的类型

```
interface EditorsProps {
    detail: string
}

// const Editors: React.FunctionComponent<props: EditorsProps> = () => {
const Editors: React.FC<props: EditorsProps> = () => {
    const { detail } = props;
    return (<></>);
};
```

Component<P,S={}>/PureComponent<P,S={}>泛型类，接收两个参数，第一个是props的类型定义，第二个是state的类型定义，例如下面的例子(但当有父组件传递属性方法或者定义state的时候还是需要,当没有的情况下省去,和不用TypeScript试用试用一样)

```
import React, { Component } from 'react'
import { connect } from 'react-redux';
import { asyncAddCount, asyncReduceCount } from '../../store/actions';
import { RouteComponentProps } from 'react-router-dom';
interface CountProps extends RouteComponentProps {//可以继承其它的接口类型
    count: number;
    asyncAddCount: (count: number) => void;
    asyncReduceCount: (count: number) => void;
}
interface CountStateType{//当需要的时候才定义
}
class Counter extends Component<CountProps, CountStateType> {
    render()：JSX.Element{
        const { count, asyncAddCount, asyncReduceCount } = this.props;
        return (
            <div>
                <h2>{count}</h2>
                <button onClick={asyncAddCount.bind(null, 10)}>Counter++</button>
                <button onClick={asyncReduceCount.bind(null, 10)}>Counter--</button>
            </div>
        )
    }
}
export default connect(
    (state: any) => ({ count: state.getIn(['countReducer', 'count']) }),
    { asyncAddCount, asyncReduceCount }
)(Counter);
```

JSX.Element return返回的jsx语法类型，例如上述的render中return的就是这个类型
类的类型就是：ComponentClass<P,S={}>泛型接口，可以在高阶组件中使用,当接收一个类或者函数的时候
```
import React, { Context,FC,ComponentClass, createContext, useReducer } from 'react';
const ProviderContext: Context<any> = createContext('provider');
export default (reducer: Function, initialState: any) => (Com: FC<any> | ComponentClass<any,any>) => {
    return () => {
        const [state, dispatch] = useReducer<any>(reducer, initialState);
        return (
            <ProviderContext.Provider value={{ state, dispatch }}>
                <Com />
            </ProviderContext.Provider >
        );
    }
}
```

Dispatch<any>泛型接口，用于定义dispatch的类型，常常用于useReducer生成的dispatch中
```
// 创建一个异步action的函数，返回一个包含异步action对象
const asyncAction = (dispatch: Dispatch<any>) => {
    return {
        asyncAddaction() {//这是一个异步的添加action,定时器模拟异步
            console.log('执行addActions之前: ' + Date.now());//打印一下时间
            setTimeout(() => {
                console.log('执行addActions : ' + Date.now());
                dispatch(addActions());//执行同步action
            }, 1000);
        }
    }
}
```

const ProviderContext: Context<any> = createContext('provider');中也可以发现，context的类型就是他的本身，一个泛型接口
```
// 源码的类型定义如下:可以发现我们需要传递一个类型，从而使得里面的参数类型也是一致
    interface Context<T> {
        Provider: Provider<T>;
        Consumer: Consumer<T>;
        displayName?: string;
    }
```

FormEvent：一个react的form表单event的类型，正常结合antd的Form表单使用
```
<form
    onSubmit={(e:FormEvent)=>{
        e.preventDefault();//取消默认事件
    }}
></form>
```

ChangeEvent: react的onChange事件触发的event类型，这是一个泛型接口，使用如下:
```
<input 
    type="text" 
    value={count} 
    onChange={(e: ChangeEvent<HTMLInputElement>) => {
       setCount(e.currentTarget.value);//HTMLInputElement表示这个一个html的input节点
    }} />
```
可选泛型类型:HTMLSelectElement、HTMLInputElement、HTMLDivElement、HTMLTextAreaElement等html标签的所有类型节点

合成事件包装器SyntheticEvent
SyntheticEvent<T = Element, E = Event>泛型接口,即原生事件的集合，就是原生事件的组合体
您的事件处理程序将传递 SyntheticEvent 的实例，这是一个跨浏览器原生事件包装器。(官方介绍)
```
  <button onClick={(e:SyntheticEvent<Element, Event>)=>{
  }}></button>
  <input onChange={(e:SyntheticEvent<Element, Event>)=>{
  }}/>
  <form
      onSubmit={(e: SyntheticEvent<Element, Event>) => {
      }}
      onBlur={(e: SyntheticEvent<Element, Event>) => {
      }}
      onKeyUp={(e: SyntheticEvent<Element, Event>) => {
      }}
  >
  </form>

```
从上面可以发现，合成事件的泛型接口在任意事件上都能适用

lazy懒加载的类型:LazyExoticComponent泛型接口,可以接受各种类型的参数,视情况而定,例如：
```
export interface RouteType {
    pathname: string;
    component: LazyExoticComponent<any>;
    exact: boolean;
    title?: string;
    icon?: string;
    children?: RouteType[];
}
export const AppRoutes: RouteType[] = [
    {
        pathname: '/login',
        component: lazy(() => import('../views/Login/Login')),
        exact: true
    },
    {
        pathname: '/404',
        component: lazy(() => import('../views/404/404')),
        exact: true,
    },
    {
        pathname: '/',
        exact: false,
        component: lazy(() => import('../views/Admin/Admin'))
    }
]
```

forwardRef，ref转发的类型定义 RefForwardingComponent泛型接口,接收两个参数
```
forwardRef(Editors) as RefForwardingComponent<any, any>;
//分别是ref的类型和props的类型,为了简单可以都定义为any
//源码类型定义如下
interface RefForwardingComponent<T, P = {}> {
    (props: PropsWithChildren<P>, ref: Ref<T>): ReactElement | null;
    propTypes?: WeakValidationMap<P>;/
    contextTypes?: ValidationMap<any>;
    defaultProps?: Partial<P>;
    displayName?: string;
}
```

MutableRefObject泛型接口，接收一个参数，作为useRef的类型定义,参数可以为T类型，即任意类型
```
const prctureRef: React.MutableRefObject<any> = useRef();
```

useState<any> hooks的useState是一个泛型函数，可以传递一个类型来定义这个hooks,当然useRef也是一个泛型函数，如果想要严谨的话也可以传递给一个类型来来定义,还有useReducer等都差不多
```
const [isShowAdd, setIsShowAdd] = useState<boolean>(false);
```

### 二、React的路由库的常用类型

RouteComponentProps: 最常见的路由api的类型定义,里面包含了history,location,match,staticContext这四个路由api的类型定义

```
import React from 'react';
import { RouteComponentProps } from 'react-router-dom';
export default function Admin({ history, location,match }: RouteComponentProps) {
    return(<>这是主页</>);
}
```

```
import React from 'react'
import { renderRoutes } from 'react-router-config'
import { RouteComponentProps } from 'react-router-dom'
import { IRoute } from '../router'

interface Props extends RouteComponentProps {
  route: IRoute
}

function BlankLayout(props: Props) {
  const { route } = props

  return <>{renderRoutes(route.routes)}</>
}

export default BlankLayout
```

### 三、axios的类型

(使用axios，往往会想到的是请求拦截和响应拦截，下面就是请求和响应的类型，以及创建一个axios的类型、还有axios错误的类型)

```
import axios, { AxiosInstance, AxiosRequestConfig, AxiosResponse,AxiosError } from 'axios'
const server: AxiosInstance = axios.create();
server.interceptors.request.use((config: AxiosRequestConfig) => {//请求拦截
    return config;
});
server.interceptors.response.use((res: AxiosResponse) => {
    if (res.status === 200) {//请求成功后 直接需要的返回数据
        res = res.data;
    }
    return res;
},(err:AxiosError)=>{});
```

### 四、Antd的类型定义

我们最熟悉的Form组件的create高阶组件，包裹组件后注入form对象，用于Form组件的使用，它的类型是FormComponentProps
```
import React from 'react';
import { Form } from 'antd';
import { FormComponentProps } from 'antd/lib/form';
interface AddFormProps extends FormComponentProps {
}
function AddForm({ form }: AddFormProps) {
    return (
        <Form></Form>
    );
}
export default Form.create()(AddForm) as any;
```
而里面的form的类型是WrappedFormUtils<any>泛型接口,正常在对form赋值的时候的定义

表格定义的columns属性的每一项的类型ColumnProps<any>,参数any正常选表格的数据数组的类型，例如user的类型是UserType则就是ColumnProps,示例如下:
```
const columns: ColumnProps<ProductType>[] = [
    {
        title: '商品名称',
        dataIndex: 'name'
    },
    {
        title: '商品详情',
        dataIndex: 'desc'
    }
]
```

级联选择的Cascader的选项的类型CascaderOptionType类型,例如:
```
import React,{useState} from 'react';
import { Cascader } from 'antd';
import { CascaderOptionType } from 'antd/lib/cascader';
const CascaderTest:FC = () => {
    const [options, setOptions] = useState<Array<CascaderOptionType>>(initialOptions);
     const loadData = (selectedOptions: CascaderOptionType[] | undefined) => {}
    return (<Cascader
         options={options}//级联的数据
         loadData={loadData}// 调用这个回调函数加载下一级列表的数据
     />);
}
```

Upload组件的文件类型:UploadFile用来定义文件对象的类型,UploadFileStatus文件类型常量(antd内部源码使用)
```
//export declare type UploadFileStatus = 'error' | 'success' | 'done' | 'uploading' | 'removed';
import React from 'react';
import { Upload, Icon } from 'antd';
import { UploadFile,UploadFileStatus } from 'antd/lib/upload/interface';
function PricturesWall():JSX.Element {
    const uploadButton = (
        <div>
            <Icon type="plus" style={{ fontSize: '25px' }} />
            <div className="ant-upload-text">图片上传</div>
        </div>
    );
    const fileList:UploadFile[] = [
        {
            uid: - index,//唯一标识
            name: img,//图片文件名
            status: DONE,//图片状态 : 已上传完成
            url: BASE_IMG_URL + img//图片地址
        }
    ];
    return(<Upload
        fileList={fileList}//所有已经上传文件对象的数组
    >
        {fileList.length > 3 ? null : uploadButton}
     </Upload>);
}
```
