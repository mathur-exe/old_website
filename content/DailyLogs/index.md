---
title: Logs
---

### December 2024
<details>
  <summary>How to use `asycnio`</summary>
    - Why is `asyncio` used →
    - `asyncio` vs `nest_asyncio` →
    - Example comparing completion fn in for AWS SageMaker Endpoint

    ```
    @llm_completion_callback()
    def complete(
        self, prompt: str, formatted: bool = False, **kwargs: Any
    ) -> CompletionResponse:
        
        # only model_kwargs being passed is temperature
        model_kwargs = {**self.model_kwargs, **kwargs}
        
        if not formatted:
            prompt = self._completion_to_prompt(prompt, self.system_prompt)
        
        request_body = self.content_handler.serialize_input(prompt, model_kwargs)
        response = self._client.invoke_endpoint(
            EndpointName=self.endpoint_name,
            Body=request_body,      # request_body: dict(prompt, model_kwargs)
            ContentType=self.content_handler.content_type,
            Accept=self.content_handler.accept,
            **self.endpoint_kwargs,
        )

        response["Body"] = self.content_handler.deserialize_output(response["Body"])
        text = self.content_handler.remove_prefix(response["Body"], prompt)

        return CompletionResponse(
            text=text,
            raw=response,
            additional_kwargs={
                "model_kwargs": model_kwargs,
                "endpoint_kwargs": self.endpoint_kwargs,
            },
        )
    ```

    ```
    @llm_completion_callback()
    async def acomplete(
        self, prompt: str, formatted: bool = False, **kwargs: Any
    ) -> CompletionResponse:
        model_kwargs = {**self.model_kwargs, **kwargs}

        if not formatted:
            prompt = self._completion_to_prompt(prompt, self.system_prompt)

        request_body = self.content_handler.serialize_input(prompt, model_kwargs)

        # Run the blocking invoke_endpoint call in a separate thread
        loop = asyncio.get_running_loop()
        partial_invoke = functools.partial(
            self._client.invoke_endpoint,
            EndpointName=self.endpoint_name,
            Body=request_body,
            ContentType=self.content_handler.content_type,
            Accept=self.content_handler.accept,
            **self.endpoint_kwargs,
        )
        response = await loop.run_in_executor(None, partial_invoke)

        response["Body"] = self.content_handler.deserialize_output(response["Body"])
        text = self.content_handler.remove_prefix(response["Body"], prompt)

        return CompletionResponse(
            text=text,
            raw=response,
            additional_kwargs={
                "model_kwargs": model_kwargs,
                "endpoint_kwargs": self.endpoint_kwargs,
            },
        )
    ```

</details> 