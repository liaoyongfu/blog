---
title: react-native中FlatList的使用
date: 2017-9-29 10:18:53
tags:
    - react-native
---
原文地址：https://medium.com/react-native-development/how-to-use-the-flatlist-component-react-native-basics-92c482816fe6

## 简介

自从0.43版本以来，react-native新增了两个新的列表视图组件：`FlatList`以及`SectionList`，现在我们来看看`FlatList`组件的使用。

## 简单示例

在`FlatList`中有个比较重要的属性：`data`和`renderItem`属性，data为object数组，renderItem可以控制每个item的渲染规则。

这边的示例我们将会从[Random User Generator API](https://randomuser.me/)中拿，UI渲染将会使用[react-native-elements](https://github.com/react-native-training/react-native-elements)的组件。

[点击查看完整源代码](https://github.com/spencercarli/react-native-flatlist-demo)


```
//FlatListDemo.js
import React, { Component } from "react";
import { View, Text, FlatList } from "react-native";

class FlatListDemo extends Component {
  constructor(props) {
    super(props);

    this.state = {
      loading: false,
      data: [],
      page: 1,
      seed: 1,
      error: null,
      refreshing: false,
    };
  }

  componentDidMount() {
    this.makeRemoteRequest();
  }

  makeRemoteRequest = () => {
    const { page, seed } = this.state;
    const url = `https://randomuser.me/api/?seed=${seed}&page=${page}&results=20`;
    this.setState({ loading: true });
    fetch(url)
      .then(res => res.json())
      .then(res => {
        this.setState({
          data: page === 1 ? res.results : [...this.state.data, ...res.results],
          error: res.error || null,
          loading: false,
          refreshing: false
        });
      })
      .catch(error => {
        this.setState({ error, loading: false });
      });
  };

  render() {
    return (
      <View style={{ flex: 1, alignItems: "center", justifyContent: "center" }}>
        <Text>Coming soon...</Text>
      </View>
    );
  }
}

export default FlatListDemo;
```

然后外边就可以使用：

```
import FlatList from './FlatListDemo.js';
import React from 'react';
import { List, ListItem } from 'react-native-elements';

class AwesomeProject extends React.Component{
    render(){
        <List>
            <FlatList
                data={this.state.data}
                renderItem={({ item }) => (
                  <ListItem
                    roundAvatar
                    title={`${item.name.first} ${item.name.last}`}
                    subtitle={item.email}
                    avatar={{ uri: item.picture.thumbnail }}
                    keyExtractor={item => item.email}
                  />
                )}
            />
        </List>
    }
}
```

效果图如下：

![demo](../../images/1-wxQVmFfaj-wSlBCL5oq2Jw.png)
