---
id: supabase
title: Supabase Data Provider Tutorial
---
import login from '@site/static/img/guides-and-concepts/data-provider/supabase/login-screen.png';
import welcome from '@site/static/img/guides-and-concepts/data-provider/supabase/welcome.png';
import flow3 from '@site/static/img/guides-and-concepts/data-provider/supabase/flow3.png';
import list from '@site/static/img/guides-and-concepts/data-provider/supabase/list.png';
import create from '@site/static/img/guides-and-concepts/data-provider/supabase/create.png';
import realTime from '@site/static/img/guides-and-concepts/data-provider/supabase/real-time.gif';
import socialLogin from '@site/static/img/guides-and-concepts/data-provider/supabase/social-login.gif';
import supabaseConfig from '@site/static/img/guides-and-concepts/data-provider/supabase/supabase-config.png';


## Introduction

Supabase is an open-source Firebase alternative that provides backend features. This tutorial steps will focus specifically on database and authentication features. We'll see how to use Supabase as a data provider and implement authentication to refine app.

refine offers built-in data provider support for Supabase and handles all required data service methods out-of-the-box. Therefore, we will not need to use complex boilerplate codes to make it work. refine handles all the complex works for us by internal hooks and implementations.

We'll build a simple CRUD app with refine and use Supabase as a data provider. We'll also see how to use Supabase's authentication features on refine app.

We are assuming that you have already know how refine works. If not, please check out the [Tutorial](https://refine.dev/docs/tutorials/ant-design-tutorial/) section first.

[Refer to docs for more information about data provider &#8594](https://refine.dev/docs/api-reference/core/providers/data-provider/)

[Discover the +15 most popular backend service data providers supported out-of-the-box by refine &#8594](https://refine.dev/integrations/)

## Project Setup

We'll be using superplate CLI to bootstrap our example project with a special preset defined to supabase example

```
npx superplate-cli --preset refine-antd-supabase my-supabase-app
```

Also, we need to install npm packages to use markdown editor:

```
npm i react-markdown react-mde
```

This will create a new refine app with supabase data provider and Ant Desing as a UI framework. We'll be using this project as a base to implement our example.

You are free to bootstrap a refine app with any other features you want. To do so, you can run the following command and choose any data provider or feature you want.

```
npx superplate-cli example-app
```

:::tip
If you want to add Supabase data provider to existed refine app, you add it by running:

```
npm i @pankod/refine-supabase
```
:::



## Establishing Supabase connection

### Initialize Supabase client

If you head over to `src/utilty` folder, you'll see a file called `supabaseClient.ts` created by CLI. This auto-generated file contains API credentials and a function that initializes the Supabase client.  

```ts
import { createClient } from "@pankod/refine-supabase";

const SUPABASE_URL = "https://iwdfzvfqbtokqetmbmbp.supabase.co";
const SUPABASE_KEY =
    "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJyb2xlIjoiYW5vbiIsImlhdCI6MTYzMDU2NzAxMCwiZXhwIjoxOTQ2MTQzMDEwfQ._gr6kXGkQBi9BM9dx5vKaNKYj_DJN1xlkarprGpM_fU";

export const supabaseClient = createClient(SUPABASE_URL, SUPABASE_KEY);
```

We'll use this example API credentials and `createClient` method that exposes from `refine-supabase` package for enabling refine to Supabase API connection.

[You can find your Supabase URL and key from your Supabase dashboard &#8594 ](https://supabase.com/docs/guides/with-react#get-the-api-keys)

:::tip
You can also use environment variables to store your Supabase URL and key. This is a good practice to keep your sensitive information safe.
:::

### Register Supabase data provider
Let's head over to `App.tsx` file where all magic happens. This is the entry point of our app. We'll be registering our Supabase data provider here.

```tsx title="App.tsx"
import { Refine } from "@pankod/refine-core";
...
 // highlight-start
import { dataProvider } from "@pankod/refine-supabase";
import { supabaseClient } from "utility";
// highlight-end

function App() {
  return (
    <Refine
      ...
      // highlight-next-line
      dataProvider={dataProvider(supabaseClient)}
      ...
    />
  );
}

export default App;
```

Highlighted lines are the ones the CLI generator automatically added to register Supabase data provider. Simply, we are passing `supabaseClient` to `dataProvider` method to establish a connection with Supabase API.

With this configuration, refine can now communicate with Supabase API and perform all required data service CRUD methods using data hooks.

[Refer to documentation to learn more about how to use data hooks &#8594](https://refine.dev/docs/api-reference/core/hooks/data-hooks/)


## Understanding the Auth Provider 

Auth provider is a concept that allows us to use any authentication service with refine. 

You'll see a file called `src/authProvider.ts` created by CLI. This auto-generated file contains pre-defined functions using Supabase Auth API methods internally to perform authentication and authorization operations. 

So basically, this is where we set complete authentication logic for the app.

Since we preferred refine-supabase as the data provider during the CLI project initialization,  all required Supabase authentication methods are already implemented for us. This shows us how easy it is to bootstrap a refine app with CLI

[Refer to docs for more information about Auth Provider methods and custom Auth Providers  &#8594](https://refine.dev/docs/api-reference/core/providers/auth-provider/)


<details><summary>Take a look the auto-generated <b>authProvider.ts</b> file </summary>
<p>


```ts title="src/authProvider.ts"
import { AuthProvider } from "@pankod/refine-core";

import { supabaseClient } from "utility";

const authProvider: AuthProvider = {
  login: async ({ email, password, providerName }) => {
    const { user, error } = await supabaseClient.auth.signIn({
      email,
      password,
      provider: providerName,
    });

    if (error) {
      return Promise.reject(error);
    }

    if (user) {
      return Promise.resolve();
    }

    // for third-party login
    return Promise.resolve(false);
  },
  register: async ({ email, password }) => {
    const { user, error } = await supabaseClient.auth.signUp({
      email,
      password,
    });

    if (error) {
      return Promise.reject(error);
    }

    if (user) {
      return Promise.resolve();
    }
  },
  forgotPassword: async ({ email }) => {
    const { data, error } = await supabaseClient.auth.api.resetPasswordForEmail(
      email,
      {
        redirectTo: `${window.location.origin}/update-password`,
      }
    );

    if (error) {
      return Promise.reject(error);
    }

    if (data) {
      return Promise.resolve();
    }
  },
  updatePassword: async ({ password }) => {
    const { data, error } = await supabaseClient.auth.update({ password });

    if (error) {
      return Promise.reject(error);
    }

    if (data) {
      return Promise.resolve("/");
    }
  },
  logout: async () => {
    const { error } = await supabaseClient.auth.signOut();

    if (error) {
      return Promise.reject(error);
    }

    return Promise.resolve("/");
  },
  checkError: () => Promise.resolve(),
  checkAuth: async () => {
    const session = supabaseClient.auth.session();
    const sessionFromURL = await supabaseClient.auth.getSessionFromUrl();

    if (session || sessionFromURL?.data?.user) {
      return Promise.resolve();
    }

    return Promise.reject();
  },
  getPermissions: async () => {
    const user = supabaseClient.auth.user();

    if (user) {
      return Promise.resolve(user.role);
    }
  },
  getUserIdentity: async () => {
    const user = supabaseClient.auth.user();

    if (user) {
      return Promise.resolve({
        ...user,
        name: user.email,
      });
    }
  },
};

export default authProvider;

```

 </p>
</details>



:::tip 
Auth provider functions are also consumed by [refine authorization hooks](https://refine.dev/docs/api-reference/core/hooks/auth/useLogin/). Since this is out of scope of this tutorial, we'll not cover them for now
:::

<br/>

Auth provider needed to be registered in `<Refine>` component to activate auth features in our app


```tsx title="App.tsx"
import { Refine } from "@pankod/refine-core";
...
 // highlight-start
import authProvider from './authProvider';
// highlight-end

function App() {
  return (
    <Refine
      ...
      // highlight-next-line
      authProvider={authProvider}
      ...
    />
  );
}

export default App;
```


Also, we'll see the `Auth provider` methods in action when using `LoginPage` in the next sections.

 
At this point, our refine app is configured to communicate with Supabase API and ready to perform authentication operations using Supabase Auth methods.

If you head over to localhost:3000, you'll see a welcome page.

<div class="img-container">
    <div class="window">
        <div class="control red"></div>
        <div class="control orange"></div>
        <div class="control green"></div>
    </div>
    <img src={welcome} alt="welcome" />
</div>

<br/>

Now it's time to add some resources to our app.


## Adding CRUD pages   
Before diving into Supabase features, we'll add simple CRUD pages to make the app more interactive.


:::note
Since this post focuses on Supabase implementation, we'll not discuss how to create CRUD pages and how it works. You can refer to [Tutorial](https://refine.dev/docs/tutorials/ant-design-tutorial/#creating-a-list-page) to learn more about creating CRUD pages.
:::

### Adding a List page

Let's add a listing page to show data retrieved from Supabase API in the table. Copy and paste the following code to `src/pages/posts` folder and name it `list.tsx`.


<details><summary>Show the List Page code</summary>
<p>

```tsx title="src/pages/posts/list.tsx"
import { IResourceComponentsProps } from "@pankod/refine-core";

import {
    List,
    Table,
    useTable,
    Space,
    EditButton,
    ShowButton,
    getDefaultSortOrder,
    FilterDropdown,
    Select,
    useSelect,
} from "@pankod/refine-antd";

import { IPost, ICategory } from "interfaces";

export const PostList: React.FC<IResourceComponentsProps> = () => {
    const { tableProps, sorter } = useTable<IPost>({
        initialSorter: [
            {
                field: "id",
                order: "asc",
            },
        ],
        metaData: {
            select: "*, categories(title)",
        },
    });

    const { selectProps } = useSelect<ICategory>({
        resource: "categories",
    });

    return (
        <List>
            <Table {...tableProps} rowKey="id">
                <Table.Column
                    key="id"
                    dataIndex="id"
                    title="ID"
                    sorter
                    defaultSortOrder={getDefaultSortOrder("id", sorter)}
                />
                <Table.Column
                    key="title"
                    dataIndex="title"
                    title="Title"
                    sorter
                />
                <Table.Column
                    key="categoryId"
                    dataIndex={["categories", "title"]}
                    title="Category"
                    defaultSortOrder={getDefaultSortOrder(
                        "categories.title",
                        sorter,
                    )}
                    filterDropdown={(props) => (
                        <FilterDropdown {...props}>
                            <Select
                                style={{ minWidth: 200 }}
                                mode="multiple"
                                placeholder="Select Category"
                                {...selectProps}
                            />
                        </FilterDropdown>
                    )}
                />
                <Table.Column<IPost>
                    title="Actions"
                    dataIndex="actions"
                    render={(_, record) => (
                        <Space>
                            <EditButton
                                hideText
                                size="small"
                                recordItemId={record.id}
                            />
                            <ShowButton
                                hideText
                                size="small"
                                recordItemId={record.id}
                            />
                        </Space>
                    )}
                />
            </Table>
        </List>
    );
};
```

 </p>
</details>



### Adding a Create page

We'll need a page for creating a new record in Supabase API. Copy and paste following code to `src/pages/posts` folder and name it `create.tsx`.

<details><summary>Show the Create Page code</summary>
<p>

```tsx title="src/pages/posts/create.tsx"
import { useState } from "react";
import { IResourceComponentsProps } from "@pankod/refine-core";

import {
    Create,
    Form,
    Input,
    Select,
    Upload,
    useForm,
    useSelect,
    RcFile,
} from "@pankod/refine-antd";

import ReactMarkdown from "react-markdown";
import ReactMde from "react-mde";

import "react-mde/lib/styles/css/react-mde-all.css";

import { IPost, ICategory } from "interfaces";
import { supabaseClient, normalizeFile } from "utility";

export const PostCreate: React.FC<IResourceComponentsProps> = () => {
    const { formProps, saveButtonProps } = useForm<IPost>();

    const { selectProps: categorySelectProps } = useSelect<ICategory>({
        resource: "categories",
    });

    const [selectedTab, setSelectedTab] = useState<"write" | "preview">(
        "write",
    );

    return (
        <Create saveButtonProps={saveButtonProps}>
            <Form {...formProps} layout="vertical">
                <Form.Item
                    label="Title"
                    name="title"
                    rules={[
                        {
                            required: true,
                        },
                    ]}
                >
                    <Input />
                </Form.Item>
                <Form.Item
                    label="Category"
                    name="categoryId"
                    rules={[
                        {
                            required: true,
                        },
                    ]}
                >
                    <Select {...categorySelectProps} />
                </Form.Item>
                <Form.Item
                    label="Content"
                    name="content"
                    rules={[
                        {
                            required: true,
                        },
                    ]}
                >
                    <ReactMde
                        selectedTab={selectedTab}
                        onTabChange={setSelectedTab}
                        generateMarkdownPreview={(markdown) =>
                            Promise.resolve(
                                <ReactMarkdown>{markdown}</ReactMarkdown>,
                            )
                        }
                    />
                </Form.Item>
                <Form.Item label="Images">
                    <Form.Item
                        name="images"
                        valuePropName="fileList"
                        normalize={normalizeFile}
                        noStyle
                    >
                        <Upload.Dragger
                            name="file"
                            listType="picture"
                            multiple
                            customRequest={async ({
                                file,
                                onError,
                                onSuccess,
                            }) => {
                                try {
                                    const rcFile = file as RcFile;
                                    await supabaseClient.storage
                                        .from("refine")
                                        .upload(`public/${rcFile.name}`, file, {
                                            cacheControl: "3600",
                                            upsert: true,
                                        });

                                    const { data } =
                                        await supabaseClient.storage
                                            .from("refine")
                                            .getPublicUrl(
                                                `public/${rcFile.name}`,
                                            );

                                    const xhr = new XMLHttpRequest();
                                    onSuccess &&
                                        onSuccess(
                                            { url: data?.publicURL },
                                            xhr,
                                        );
                                } catch (error) {
                                    onError &&
                                        onError(new Error("Upload Error"));
                                }
                            }}
                        >
                            <p className="ant-upload-text">
                                Drag & drop a file in this area
                            </p>
                        </Upload.Dragger>
                    </Form.Item>
                </Form.Item>
            </Form>
        </Create>
    );
};
```

 </p>
</details>

### Adding an Edit page

We'll need a page for editing a record in Supabase API. Copy and paste following code to `src/pages/posts` folder and name it `edit.tsx`.

<details><summary>Show the Edit Page code</summary>
<p>

```tsx title="src/pages/posts/edit.tsx"
import React, { useState } from 'react';
import { IResourceComponentsProps } from '@pankod/refine-core';

import {
    Alert,
    Button,
    Edit,
    Form,
    Input,
    ListButton,
    RcFile,
    RefreshButton,
    Select,
    Upload,
    useForm,
    useSelect,
} from '@pankod/refine-antd';

import ReactMarkdown from 'react-markdown';
import ReactMde from 'react-mde';

import 'react-mde/lib/styles/css/react-mde-all.css';

import { IPost, ICategory } from 'interfaces';
import { supabaseClient, normalizeFile } from 'utility';

export const PostEdit: React.FC<IResourceComponentsProps> = () => {
    const [isDeprecated, setIsDeprecated] = useState(false);
    const { formProps, saveButtonProps, queryResult } = useForm<IPost>({
        liveMode: 'manual',
        onLiveEvent: () => {
            setIsDeprecated(true);
        },
    });

    const postData = queryResult?.data?.data;
    const { selectProps: categorySelectProps } = useSelect<ICategory>({
        resource: 'categories',
        defaultValue: postData?.categoryId,
    });

    const [selectedTab, setSelectedTab] = useState<'write' | 'preview'>(
        'write'
    );

    const handleRefresh = () => {
        queryResult?.refetch();
        setIsDeprecated(false);
    };

    return (
        <Edit
            saveButtonProps={saveButtonProps}
            pageHeaderProps={{
                extra: (
                    <>
                        <ListButton />
                        <RefreshButton onClick={handleRefresh} />
                    </>
                ),
            }}
        >
            {isDeprecated && (
                <Alert
                    message="This post is changed. Reload to see it's latest version."
                    type="warning"
                    style={{
                        marginBottom: 20,
                    }}
                    action={
                        <Button
                            onClick={handleRefresh}
                            size="small"
                            type="ghost"
                        >
                            Refresh
                        </Button>
                    }
                />
            )}

            <Form {...formProps} layout="vertical">
                <Form.Item
                    label="Title"
                    name="title"
                    rules={[
                        {
                            required: true,
                        },
                    ]}
                >
                    <Input />
                </Form.Item>
                <Form.Item
                    label="Category"
                    name="categoryId"
                    rules={[
                        {
                            required: true,
                        },
                    ]}
                >
                    <Select {...categorySelectProps} />
                </Form.Item>
                <Form.Item
                    label="Content"
                    name="content"
                    rules={[
                        {
                            required: true,
                        },
                    ]}
                >
                    <ReactMde
                        selectedTab={selectedTab}
                        onTabChange={setSelectedTab}
                        generateMarkdownPreview={(markdown) =>
                            Promise.resolve(
                                <ReactMarkdown>{markdown}</ReactMarkdown>
                            )
                        }
                    />
                </Form.Item>
                <Form.Item label="Images">
                    <Form.Item
                        name="images"
                        valuePropName="fileList"
                        normalize={normalizeFile}
                        noStyle
                    >
                        <Upload.Dragger
                            name="file"
                            listType="picture"
                            multiple
                            customRequest={async ({
                                file,
                                onError,
                                onSuccess,
                            }) => {
                                const rcFile = file as RcFile;
                                const fileUrl = `public/${rcFile.name}`;

                                const { error } = await supabaseClient.storage
                                    .from('refine')
                                    .upload(fileUrl, file, {
                                        cacheControl: '3600',
                                        upsert: true,
                                    });

                                if (error) {
                                    return onError?.(error);
                                }
                                const { data, error: urlError } =
                                    await supabaseClient.storage
                                        .from('refine')
                                        .getPublicUrl(fileUrl);

                                if (urlError) {
                                    return onError?.(urlError);
                                }

                                onSuccess?.(
                                    { url: data?.publicURL },
                                    new XMLHttpRequest()
                                );
                            }}
                        >
                            <p className="ant-upload-text">
                                Drag & drop a file in this area
                            </p>
                        </Upload.Dragger>
                    </Form.Item>
                </Form.Item>
            </Form>
        </Edit>
    );
};
```

 </p>
</details>


### Adding Interfaces and Normalize file

We need to add interfaces for `Post` and `Create` pages to `src/interfaces/index.d.ts` file.

<details><summary>Show the interface code </summary>
<p>

```tsx title="src/interfaces/index.d.ts"
export interface ICategory {
    id: string;
    title: string;
}

export interface IFile {
    name: string;
    percent: number;
    size: number;
    status: 'error' | 'success' | 'done' | 'uploading' | 'removed';
    type: string;
    uid: string;
    url: string;
}

export interface IPost {
    id: string;
    title: string;
    content: string;
    categoryId: string;
    images: IFile[];
}
```

 </p>
</details>

Also, the `normalizeFile` function needed to be added to the `src/utility/normalize.ts` file to perform file upload operations specifically for Supabase API.

<details><summary>Show the Normalize file code</summary>
<p>

```tsx title="src/utility/normalize.ts"
import { UploadFile } from "@pankod/refine-antd";

interface UploadResponse {
    url: string;
}
interface EventArgs<T = UploadResponse> {
    file: UploadFile<T>;
    fileList: Array<UploadFile<T>>;
}

export const normalizeFile = (event: EventArgs) => {
    const { fileList } = event;

    return fileList.map((item) => {
        const { uid, name, type, size, response, percent, status } = item;

        return {
            uid,
            name,
            url: item.url || response?.url,
            type,
            size,
            percent,
            status,
        };
    });
};
```

</p>
</details>

### Adding Resources
One last thing we need to do is to add newly created CRUD pages to the `resources` property of `<Refine>` component. 

```tsx title="src/App.tsx"
import { dataProvider } from '@pankod/refine-supabase';
import { supabaseClient } from 'utility';
//highlight-next-line
import { PostList, PostCreate, PostEdit } from 'pages/posts';

function App() {
    return (
        <Refine
            ...
            dataProvider={dataProvider(supabaseClient)}
            //highlight-start
            resources={[
                {
                    name: 'posts',
                    list: PostList,
                    create: PostCreate,
                    edit: PostEdit
                },
            ]}
            //highlight-end
        />
    );
}

export default App;
```


The resources property activates the connection between CRUD pages and Supabase API. 

**refine** automatically matches the Supabase API endpoint with CRUD pages for us. In this way, the pages can interact with data from the API.

- The `name` property refers to the name of the table in the Supabase database.

- The `list` property registers `/posts` endpoint to the `PostList` component.

- The `create` property registers `/posts/create` endpoint to the `PostCreate` component. Thereby, when you head over to `yourdomain.com/posts/create`, you will see the `PostCreate` page you just created.


[Refer to resources docs for more information &#8594](https://refine.dev/docs/api-reference/core/components/refine-config/#resources)



## Understanding the Login screen 
After adding the resources, the app will look like:


<div class="img-container">
    <div class="window" style={{alignSelf:"center", width:"700px"}} >
        <div class="control red"></div>
        <div class="control orange"></div>
        <div class="control green"></div>
    </div>
    <img style={{alignSelf:"center", width:"700px"}} src={login} alt="login" />
</div>


<br/>

Normally, refine shows a default login page when `authProvider` and `resources` properties are passed to `<Refine />` component. However, our login screen is slightly different from the default one.

#### This premade and ready to use Login screen consist `LoginPage` and `authProvider` concepts behind the scenes:



Let's check out the `LoginPage` property:

  ```tsx title="src/App.tsx"
import { Refine } from '@pankod/refine-core';
//highlight-start
import { AuthPage } from '@pankod/refine-antd';
import routerProvider from "@pankod/refine-react-router-v6";
  //highlight-end
import authProvider from './authProvider';
...

function App() {
    return (
        <Refine
            ...
            //highlight-start
             routerProvider={{
                ...routerProvider,
                routes: [
                    {
                        path: '/register',
                        element: <AuthPage type="register" />,
                    },
                    {
                        path: '/forgot-password',
                        element: <AuthPage type="forgotPassword" />,
                    },
                    {
                        path: '/update-password',
                        element: <AuthPage type="updatePassword" />,
                    },
                ],
            }}
            //highlight-end
            authProvider={authProvider}
            //highlight-start
            LoginPage={AuthPage}
            //highlight-end
        />
    );
}
```
The `AuthPage` component returns ready-to-use authentication pages for login, register, update, and forgot password actions. We passed it to the `LoginPage` property to override the default login page.

**This is where `authProvider` comes into play.** 

Remember the [Understanding the Auth Provider](#understanding-auth-provider) section? We mentioned "login", "register," and "forgotPassword"  functions that use Supabase Auth API methods internally in the `authProvider.ts` file. These methods automatically bind to `<AuthPage>` components by **refine** to perform authentication operations.




<div class="img-container">
     <img style={{alignSelf:"center", width:"700px"}} src={flow3} alt="flow3" />
</div>

<br/>
<br/>

**By defining the routes array in the `routerProvider` property, we can access the `<AuthPage>` authentication pages by navigating to `/register`, `/forgot-password`, and `/update-password` endpoints.**

We'll show how to implement third party logins in the next sections.

[Refer to AuthPage docs for more information &#8594](https://refine.dev/docs/api-reference/antd/components/auth-page) 



Sign in the app with followings credentials:


- email: info@refine.dev
- password: refine-supabase

We have successfully logged in to the app and `ListPage` renders table of data at the `/post` route.

<div class="img-container">
    <div class="window">
        <div class="control red"></div>
        <div class="control orange"></div>
        <div class="control green"></div>
    </div>
    <img src={list} alt="list" />
</div>

<br/>
<br/>

Now click on the `Create` button to create a new post. The app will navigate to the `post/create` endpoint, and `CreatePage` will render.

Thanks to `refine-supabase` data provider, we can now start creating new records for the Supabase Database by just filling the form.

<div class="img-container">
    <div class="window">
        <div class="control red"></div>
        <div class="control orange"></div>
        <div class="control green"></div>
    </div>
    <img src={create} alt="create" />
</div>

<br/>

## Let's recap what we have done so far

So far, we have implemented the followings:
- We have reviewed Supabase Client and data provider concepts. We've seen benefits of using **refine** and how it can handle complex setups for us.
- We have talked about the `authProvider` concept and how it works with Supabase Auth API. We also see the advantages of **refine**'s built-in authentication support.
- We have added CRUD pages to make the app interact with Supabase API. We've seen how the `resources` property works and how it connects the pages with the API.
- We have seen how the `LoginPage` property works and how it overrides the default login page with the `AuthPage` component. We also saw how the `authProvider` property works with the `AuthPage` component.
- We have seen how authorization handling in **refine** app by understanding the logic behind of  `LoginPage` property, `authProvider`, and `<AuthPage>` component. 


**refine provides solutions for critical parts of the complete CRUD app requirements. It saves development time and effort by providing ready-to-use components and features.**

## Supabase Real Time Support

refine has a built-in support for [Supabase Real Time](https://supabase.com/docs/guides/realtime). It means that when you create, update, or delete a record, the changes will be reflected in the app in real-time.  
Required Supabase Real Time setup is already done in the [`@pankod/refine-supabase`](https://github.com/pankod/refine/tree/master/packages/supabase)` data provider. 

 [You can check the Supabase Real Time integration in the data provider source code  &#8594](https://github.com/pankod/refine/blob/master/packages/supabase/src/index.ts#L325)

We only need to register refine's Supabase Live Provider to the `liveProvider` property to enable real-time support.

```tsx title="src/App.tsx"
import { Refine } from '@pankod/refine-core';
//highlight-start
import { liveProvider } from "@pankod/refine-supabase";
import { supabaseClient } from 'utility';
//highlight-end
...

function App() {
    return (
        <Refine
            ...
            //highlight-start
             liveProvider={liveProvider(supabaseClient)}
             options={{ liveMode: "auto" }}
            //highlight-end
            ...
        />
    );
}
```
:::note
For live features to work automatically, we setted `liveMode: "auto"` in the options prop.

[Refer to Live Provider docs for more information &#8594](https://refine.dev/docs/api-reference/core/providers/live-provider/#livemode)
:::

<br/>

### Let see how Real Time works in the app

<br/>

<div class="img-container">
    <img  src={realTime} alt="realTime" />
</div>


<br/>



:::tip
**refine** offers out-of-the-box live provider support:

-   **Ably** &#8594 [Source Code](https://github.com/pankod/refine/blob/master/packages/ably/src/index.ts) - [Demo](https://stackblitz.com/github/pankod/refine/tree/master/examples/ably/?preset=node)
-   **Supabase** &#8594 [Source Code](https://github.com/pankod/refine/blob/master/packages/supabase/src/index.ts#L187)
-   **Appwrite** &#8594 [Source Code](https://github.com/pankod/refine/blob/master/packages/appwrite/src/index.ts#L252)
-   **Hasura** &#8594 [Source Code](https://github.com/pankod/refine/blob/master/packages/hasura/src/liveProvider/index.ts#L16)
-   **Nhost** &#8594 [Source Code](https://github.com/pankod/refine/blob/master/packages/nhost/src/liveProvider/index.ts#L16)
:::


## Bonus Section: Login with Google

We'll show how to add Google Login option to the app. 

Social login feature can be activated by setting `provider` property of the `<AuthPage>` component.

?? avaiable providers?? 

```tsx title="src/App.tsx"
import { Refine } from '@pankod/refine-core';
//highlight-start
import { AuthPage } from '@pankod/refine-antd';
import { GoogleOutlined } from "@ant-design/icons";
  //highlight-end
...

function App() {
    return (
        <Refine
            ...
            //highlight-start
            LoginPage={() => (
                <AuthPage
                    type="login"
                    providers={[
                        {
                            name: "google",
                            label: "Sign in with Google",
                            icon: (
                                <GoogleOutlined
                                    style={{ fontSize: 18, lineHeight: 0 }}
                                />
                            ),
                        },
                    ]}
                />
            )}
            //highlight-end
        />
    );
}
```

This will add a new Google login button to the login page.  After the user successfully logs in, the app will redirect back to the app.

### Enable Google Auth on Supabase

Head over to app.supabase.com and sign in to your Supabase account. Next, go to Authentication -> Settings to configure the Auth providers.

You will find the Google Auth option in the Auth providers section; enable it and set your Google Credentials.

[Refer to Supabase docs for more information about Credentials  &#8594](https://supabase.com/docs/guides/auth/auth-google#create-your-google-credentials)


<div class="img-container">
    <div class="window">
        <div class="control red"></div>
        <div class="control orange"></div>
        <div class="control green"></div>
    </div>
    <img src={supabaseConfig} alt="supabaseConfig" />
</div>

<br/>


Here is the result:

<br/>

<div class="img-container">
    <div class="window">
        <div class="control red"></div>
        <div class="control orange"></div>
        <div class="control green"></div>
    </div>
    <img src={socialLogin} alt="socialLogin" />
</div>

<br/>