# Member Portal Data Architecture

Preface: Designing a system is quite an opinionated topic and there are multiple ways to go about implementing one. In this section, we'll cover how this project goes through the process of reading data from the AWS Cloud and renders it to the screen efficiently.

### Overview

Let's work backwards from the data to it getting rendered on the screen. Data for the member portal users and other miscellaneous information is stored in `dynamdb`. Interfacing with the `aws-sdk` and reading from dynamodb is done only by the API endpoints on our nextjs application. Each API endpoint is responsible for reading from one item from the database and returning that information. To call any of the APIs, it requires the user to be authenticated and have a valid session with `next-auth`. On the front-end, there are recoil selectors associcated with each API endpoint. When the selector is triggered, it reads dependent information from other atoms / selectors and uses that information to call the API. All recoil selectors currently are async since they depend on an API response for populating the data. On the front-end, the data is only fetched if the user navigates to a page that requires the recoil selector. The usage of `useRecoilValue` triggers the selector to fetch information and save it for later use. After having fetching the data once, recoil persists the data and will not make multiple API calls each time the user navigates to different pagse. When reading data its important to note that the value returned by `useRecoilValue` is read-only. Since our selectors are async, we cannot immediately render the data to the screen. Instead, we need to wait for the data to be fetched. To do this, we use `React Suspense`. The `React Suspense` component is a wrapper around the componenet that uses `useRecoilValue` which contains data returnde by the selector. The `React Suspense` component will render the data if it is available. If the data is not available, it will render a loading indicator. 