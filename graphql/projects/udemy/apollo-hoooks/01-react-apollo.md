---
projectType: tutorial
source: https://www.udemy.com/course/graphql-by-example/learn/lecture/16580146#overview
instructor: Mirko Nasato
title: React Apollo - 3 Different Packages
details:
  section: 7
  lesson: 44
---



# React Apollo - 3 Different Packages



**NOTE:** as of version 3 the react-apollo package has been deprecated 



> ⚠️ **THIS PROJECT HAS BEEN DEPRECATED** ⚠️
>
> Please note that 4.0.0 is the final version of all React Apollo packages. React Apollo functionality is now directly available from `@apollo/client` >= 3. While using the `@apollo/react-X` packages will still work, we recommend using the following imports from `@apollo/client` directly instead:
>
> > - old: `@apollo/react-components` --> new: `@apollo/client/react/components`
> > - old: `@apollo/react-hoc` --> new: `@apollo/client/react/hoc`
> > - old: `@apollo/react-ssr` --> new: `@apollo/client/react/ssr`
> > - old: `@apollo/react-testing` --> new: `@apollo/client/testing`
> > - old: `@apollo/react-hooks` --> new: `@apollo/client`
>
> Moving forward, all Apollo + React issues / pull requests should be opened in the [apollo-client](https://github.com/apollographql/apollo-client.git) repo. Please refer to the [Apollo Client migration guide](https://www.apollographql.com/docs/react/migrating/apollo-client-3-migration/) for more details.





React-Apollo provides utilities that can be used specifically for react applications. Thus far all of our mutations and queries have been framework agnostic we can use the same code for many different frameworks including, react, angular, and vue among others. All of these use apollo-client under the hook and provide utilties to access our data.



There have been three prior implementations of the apollo-client for react, the first used a `HOC` patter to fetch our data, then there was the `renderProps` pattern and currently in the latest rendition we have `apollo-hooks`