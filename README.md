# serverless-neg-cf

Repository containing a first approach on the usage of [Serverless NEGs](https://cloud.google.com/load-balancing/docs/negs/serverless-neg-concepts) in conjunction with Cloud Functions.

This combination will allow us to make our functions globally available, serving content from the region closer to the user and all of them behind the same endpoint that the Load Balancer will be providing.
