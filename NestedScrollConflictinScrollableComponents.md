### Nested Scroll Conflict in Scrollable Components

#### Scrollable Components:

WaterFlow, List, Grid, Scroll, Swiper

#### Scene Description:

When a Scroll component wraps a WaterFlow component:

* **Scrolling Up:** If the WaterFlow component hasn't reached the top, the entire Scroll component scrolls first.
* When the WaterFlow reaches the top, the WaterFlow component starts scrolling internally.
* **Scrolling Down:** If the WaterFlow component hasn’t reached the top, both the Scroll and WaterFlow components scroll smoothly down together.

**Code:**

```ArkTS
@Entry
@ComponentV2
struct Index {
  @Local dataArr: Array<string> = []; // Data source

  aboutToAppear(): void {
    for (let i = 0; i < 40; i++) {
      this.dataArr.push(`data_${i}`); // Adding data to the array
    }
  }

  build() {
    Column() {
      List() { // Outer scroll component
        ListItem() {
          Row()
            .width('100%')
            .height('30%')
            .backgroundColor(Color.Green)
        }

        ListItem() {
          List() { // Inner scroll component
            Repeat<string>(this.dataArr) // Looping through the data
              .each((ri: RepeatItem<string>) => {
                ListItem() {
                  Text('each_' + ri.item).fontSize(30)
                }
              })
          }
          .height('100%')
          .width('100%')
        }
      }
      .height('100%')
      .width('100%')
    }
  }
}
```

**Component Tree**

![Component Tree](https://i-blog.csdnimg.cn/direct/5b878fd52a10475789ff517ac11baa40.png)

**Effect:**

![Effect](https://i-blog.csdnimg.cn/direct/7fd3ab5f672949f49f5e138356290693.gif)

#### Solution:

Setting the **nestedScroll** mode for the WaterFlow component allows smooth scroll switching when it reaches the top.

| Enum Value Name | Description                                                                                                                                                                                                               |
| --------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| SELF\_FIRST     | The component scrolls first. After it reaches the edge, the parent component scrolls. If the parent component has an edge effect, it triggers the parent's edge effect; otherwise, it triggers the child's edge effect.   |
| PARENT\_FIRST   | The parent component scrolls first. After the parent reaches the edge, the component itself scrolls. If the component reaches the edge, it triggers its own edge effect; otherwise, it triggers the parent's edge effect. |

**Core Code:**

```ArkTS
List() {
  //...
}
.nestedScroll({
  scrollForward: NestedScrollMode.PARENT_FIRST, // Scroll Up
  scrollBackward: NestedScrollMode.SELF_FIRST // Scroll Down
})
```

**Full Code:**

```ArkTS
// Using Repeat inside List container component
@Entry
@ComponentV2
  // It is recommended to use the V2 decorator
struct Index {
  @Local dataArr: Array<string> = []; // Data source

  aboutToAppear(): void {
    for (let i = 0; i < 40; i++) {
      this.dataArr.push(`data_${i}`); // Adding data to the array
    }
  }

  build() {
    Column() {
      List() { // Outer scroll component
        ListItem() {
          Row()
            .width('100%')
            .height('30%')
            .backgroundColor(Color.Green)
        }

        ListItem() {
          List() { // Inner scroll component
            Repeat<string>(this.dataArr)
              .each((ri: RepeatItem<string>) => {
                ListItem() {
                  Text('each_' + ri.item).fontSize(30)
                }
              })
          }
          .nestedScroll({
            scrollForward: NestedScrollMode.PARENT_FIRST,
            scrollBackward: NestedScrollMode.SELF_FIRST
          })
          .height('100%')
          .width('100%')
        }
      }
      .nestedScroll({
        scrollForward: NestedScrollMode.PARENT_FIRST,
        scrollBackward: NestedScrollMode.SELF_FIRST
      })
      .height('100%')
      .width('100%')
    }
  }
}
```

**Effect After Fix:**


![Effect After Fix](https://i-blog.csdnimg.cn/direct/12276bd0d184463497623b5f1e5949d9.gif)
