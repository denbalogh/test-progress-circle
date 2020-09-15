# ProgressCircle Test

## Project setup and launch
```js
npm install

// wait for it

npm run serve
```

## ProgressCircle Component 
source file is ->[HERE](https://github.com/denbalogh/test-progress-circle/blob/master/src/components/ProgressCircle.vue)<-
| Prop                    | Type    | Default | Required | Description                                                                                                                                                                                                                                                                           |
|:----------------------------|:--------|:--------|:--------|:--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| label           | String | `''` | false | Optional label under the progress numerical values.
| size     | Enum ( `default`, `compact`)  |    `default`     | false | Size of the progress circle.
| currentValue                    | Number |    | true | Current value of the progress. 
| maxValue                    | Number |    | true | Maximum value of the progress. 

## Examples
```js
<ProgressCircle :current-value="0" :max-value="100" />
```
![alt text](https://user-images.githubusercontent.com/23406415/93207646-8c878e00-f75b-11ea-8956-0b6af50a05aa.png)

```js
<ProgressCircle :current-value="39" :max-value="100" />
```
![alt text](https://user-images.githubusercontent.com/23406415/93207635-88f40700-f75b-11ea-9931-545151566fff.png)

```js
<ProgressCircle :current-value="8" :max-value="15" :label="'Label'" />
```
![alt text](https://user-images.githubusercontent.com/23406415/93207638-8abdca80-f75b-11ea-8e9e-f724181994f1.png)

```js
<ProgressCircle :size="'compact'" :current-value="0" :max-value="100" />
```
![alt text](https://user-images.githubusercontent.com/23406415/93207644-8beef780-f75b-11ea-8863-72441a29e440.png)

```js
<ProgressCircle :size="'compact'" :current-value="42" :max-value="100" />
```
![alt text](https://user-images.githubusercontent.com/23406415/93207645-8beef780-f75b-11ea-94e5-fbfbc6ee6cfb.png)

```js
<ProgressCircle :label="'Label'" :size="'compact'" :current-value="42" :max-value="73" />
```
![alt text](https://user-images.githubusercontent.com/23406415/93207639-8b566100-f75b-11ea-816d-abe28be1bf5c.png)

## Animated transition when `compact` size
![alt text](https://user-images.githubusercontent.com/23406415/93210291-a0cd8a00-f75f-11ea-8993-258510ca1db9.gif)

## What is missing
- font family (wasn't defined and couldn't find it here https://www.whatfontis.com/)
- default size progress isn't animated when `currentValue` is changed
