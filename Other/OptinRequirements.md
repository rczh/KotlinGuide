# Opt-in Requirements
注意，opt-in注解@RequiresOptIn和@OptIn是实验性质的。@RequiresOptIn和@OptIn注解在1.3.70版本中被引入用来代替之前使用的@Experimental和@UseExperimental注解，同时，-Xopt-in编译选项用来代替-Xuse-experimental

Kotlin标准库提供了一种要求并且明确同意使用某些api元素的机制，该机制允许库开发人员将api中需要opt-in的特定条件告知用户，例如，一个API处于实验状态并且将来可能会改变

为了防止潜在的问题，编译器会就这些条件向此类api的用户发出警告，并要求他们在使用API之前进行选择

## Opting in to using API


