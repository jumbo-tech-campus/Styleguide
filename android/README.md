# Android Style Guide

This style guide follows following

## Table Of Contents

## Naming

- [Swift Style Guide](#swift-style-guide)
  * [Table Of Contents](#table-of-contents)
  * [1. Source file](#1-source-file)
    + [1.1 File names](#11-file-names)
  * [2. File structure](#2-file-structure)
  * [3. Naming](#3-naming)
    + [3.1 General](#31-general)
  * [4. Coding Style](#4-coding-style)
    + [4.1 General formatting](#41-general-formatting)
    + [4.2 Error Handling](#42-error-handling)
        + [4.2.1 Default Error Handling](#421-default-error-handling)
        + [4.2.2 Error Layout with Customized Click Event](#422-error-layout-with-customized-click-event)
        + [4.2.3 Multiple Error Layouts with Customized Click Event](#423-multiple-error-layouts-with-customized-click-event)

### 4.1 General Formatting


### 4.2 Error Handling

Every view tend to return a default error when something is wrong with the request. In some cases you may need multiple error states for the views.

#### 4.2.1 Default Error Handling

If you need only 1 error layout in return you should implement it as follows :

Into your ***XML***
```XML 
<include
    layout="@layout/error_state"
    app:errorLayoutObject="@{viewModel.networkErrorLayout}" />
```
Into your ***ViewModel***
```Kotlin
// this is the object where you should define mainText, subText, buttonText and onClick for the button.
val networkErrorLayout = ErrorLayout(
        mainText = R.string.error_state_main_text,
        subText = R.string.error_state_sub_text,
        buttonText = R.string.error_state_button_text)
{
    getPromotionsOverview()     // this code block will run when user clicks the button
}

...
// this is an example request to get some data
private suspend fun getPromotionsOverview() {
    networkErrorLayout.visibility.value = false
    promotionsRepository.getPromotionsOverview()
            .onSuccess {
                _sections.postValue(it.sections)
                // this is needed for hiding the error layout
                networkErrorLayout.visibility.value = false  
            }
            .onNetworkFailure {
                // this is needed for showing the error layout
                networkErrorLayout.visibility.value = true  
            }
}

```

#### 4.2.2 Error Layout with Customized Click Event

If you need Android context on your `onClick function then you have to implement as follows

You will `include` the layout to your `XML` same as the previous section. 

To implement a custom click you need a `SingleLiveEvent` mechanism to handle.

Into your ***ViewModel***
```Kotlin
// this object will be observed from your Fragment
val onClickEvents = SingleLiveEvent<ClickEvent>()

// this is the event you will send from viewModel
enum class ClickEvent {
    CLICK_NETWORK_ERROR
}

// this is the object where you should define mainText, subText, buttonText and onClick for the button.
val networkErrorLayout = ErrorLayout(
        mainText = R.string.error_state_main_text,
        subText = R.string.error_state_sub_text,
        buttonText = R.string.error_state_button_text)
{
    onClickEvents.value = ClickEvent.CLICK_NETWORK_ERROR    // this code block will run when user clicks the button
}

```
Into your fragment ***Fragment***

```Kotlin
viewModel.onClickEvents.observeNonNull(viewLifecycleOwner) {
    if (it == PromotionDetailViewModel.ClickEvent.CLICK_NETWORK_ERROR) {
        requireFragmentManager().popBackStack()
    }
}

```

#### 4.2.3 Multiple Error Layouts with Customized Click Event


You may need a different Error layouts for the views. In the following example the page has 2 different types of error. Which are `UNKNOWN` and `INVALID_SEARCH_FILTER` therefore you need 2 different click events which are `CLICK_NETWORK_ERROR` and `CLICK_PROMOTION_UNAVAILABLE_OR_PRODUCT_LIST_EMPTY`

Into your ***XML***
```XML 
<include
    layout="@layout/error_state"
    app:errorLayoutObject="@{viewModel.networkErrorLayout}" />
    
<include
    layout="@layout/error_state"
    app:errorLayoutObject="@{viewModel.unavailableErrorLayout}" />
```

Into your ***view model***
```Kotlin
// this object is observed from your Fragment to determine the which button is clicked
val onClickEvents = SingleLiveEvent<ClickEvent>()

// define your click objects
enum class ClickEvent {
    CLICK_NETWORK_ERROR,
    CLICK_PROMOTION_UNAVAILABLE_OR_PRODUCT_LIST_EMPTY
}

// this is for the network errors
val networkErrorLayout = ErrorLayout(
        mainText = R.string.error_state_main_text,
        subText = R.string.error_state_sub_text,
        buttonText = R.string.error_state_button_text)
{
    onClickEvents.value = ClickEvent.CLICK_NETWORK_ERROR
}

// this is for the custom error
val unavailableErrorLayout = ErrorLayout(
        mainText = R.string.error_state_detail_main_text,
        subText = R.string.error_state_detail_sub_text,
        buttonText = R.string.error_state_detail_button_text)
{
    onClickEvents.value = ClickEvent.CLICK_PROMOTION_UNAVAILABLE_OR_PRODUCT_LIST_EMPTY
}
```
Into your fragment ***Fragment***

```Kotlin
viewModel.onClickEvents.observeNonNull(viewLifecycleOwner) {
    if (it == PromotionDetailViewModel.ClickEvent.CLICK_NETWORK_ERROR) {
        // do something
    }else if (it == PromotionDetailViewModel.ClickEvent.CLICK_PROMOTION_UNAVAILABLE_OR_PRODUCT_LIST_EMPTY){
        // do something different
    }
}
```

