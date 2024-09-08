---
title: Home Assistant 接入 ChatGPT
date: 2024-09-08T19:00:00+08:00
tags:
  - 智能家居
categories:
  - ""
draft: false
---
在 Home Assistant 中，集成了 OpenAI Conversation 扩展，可以利用 API 来将自然语言来与 Home Assistant 进行交互，来更加智能地控制接入到 Home Assistant 的设备，在安装好扩展后，首先需要配置集成的 API 秘钥，接着在用户界面中，可以配置扩展的配置：
 - 模型：对应 GPT 的语言模型，比如：`gpt-3.5-turbo`，`gpt-4o`
 - 响应中返回的最大令牌数：AI 模型生成输出的最大 tokens 数量
 - Top P：取值越大，生成的随机性越高；取值越低，生成的确定性越高
 - 温度：取值越大，生成结果更加多样化；取值越低，生成结果更加确定

 在控制 Home Assistant 这个选项中选择「Assist」，否则它便无法控制设备。

 由于在国内想要访问 OpenAI 的接口比较不方便，因此可以配置其他兼容 OpenAI API 的服务，比如阿里云的模型服务灵积 DashScope，开通服务后，可以在「管理中心 - API-KEY 管理」中创建新的 API-KEY，创建成功后，可以在 OpenAI Conversation 中创建新的服务，将创建好的 API KEY 粘贴提交，在阿里云模型中心里，可以找到通义千问的模型名称，在这里使用 `qwen-plus`，点击保存:

![截屏2024-09-08 19.12.38.png](https://image-1301539196.cos.ap-guangzhou.myqcloud.com/%E6%88%AA%E5%B1%8F2024-09-08%2019.12.38.png)

然后在 Home Assistant 的「语音助手」设置中，可以添加语音助手，在对话代理中勾选 OpenAI Conversation，完成创建。

在 lovelace 中弹出聊天框可以跟语音助手进行对话，这时候输入「打开客厅的灯」，客厅区域的灯光随之被打开，然后语音助手回复「客厅的灯已经打开了。」：

![截屏2024-09-08 19.40.47.jpg](https://image-1301539196.cos.ap-guangzhou.myqcloud.com/%E6%88%AA%E5%B1%8F2024-09-08%2019.40.47.jpg)

在 OpenAI Conversation 扩展配置中，默认的指示模板如下，在与 OpenAI API 进行交互时，它会作为 Prompt 的一部分发送给模型服务，在这里告诉了模型服务在家庭中的区域以及设备有哪些，来让模型更好地区理解用户的输入指令：

```
This smart home is controlled by Home Assistant.

An overview of the areas and the devices in this smart home:
{%- for area in areas() %}
  {%- set area_info = namespace(printed=false) %}
  {%- for device in area_devices(area) -%}
    {%- if not device_attr(device, "disabled_by") and not device_attr(device, "entry_type") and device_attr(device, "name") %}
      {%- if not area_info.printed %}

{{ area_name(area) }}:
        {%- set area_info.printed = true %}
      {%- endif %}
- {{ device_attr(device, "name") }}{% if device_attr(device, "model") and (device_attr(device, "model") | string) not in (device_attr(device, "name") | string) %} ({{ device_attr(device, "model") }}){% endif %}
    {%- endif %}
  {%- endfor %}
{%- endfor %}

Answer the user's questions about the world truthfully.

If the user wants to control a device, reject the request and suggest using the Home Assistant app.
```

打开 OpenAI Conversation 的调试日志，开启新的一轮对话：

![截屏2024-09-08 19.53.02.png](https://image-1301539196.cos.ap-guangzhou.myqcloud.com/%E6%88%AA%E5%B1%8F2024-09-08%2019.53.02.png)

回到配置中，点击禁用调试日志，会生成一份调试日志，观察调试日志可以发现，Home Assistant 发送给模型的 Prompt 中，包含了一些除了指示定义的 system 预设输入，比如当前时间，可用的设备列表（包括设备名称，类型和当前状态），以及当接收用户输入来控制 Home Assistant 时，应该去调用意图工具（intent tool），这个列表列出了模型可以调用的 tools，目前主要是函数，模型可以生成用于这些函数的输入参数的 JSON 格式，日志里还打印了 tools 的列表，其中包括了 `HassTurnOn` 和 `HassTurnOff` 两个功能的定义，以及接收的参数：其中控制设备的主要参数是 `name` 和 `domain`：

```
2024-09-08 11:52:29.732 DEBUG (MainThread) [homeassistant.components.openai_conversation] Prompt: [{'role': 'system', 'content': "Current time is 19:52:29. Today's date is 2024-09-08.\nYou are a voice assistant for Home Assistant.\nAnswer questions about the world truthfully.\nAnswer in plain text. Keep it simple and to the point.\nWhen controlling Home Assistant always call the intent tools. Use HassTurnOn to lock and HassTurnOff to unlock a lock. When controlling a device, prefer passing just name and domain. When controlling an area, prefer passing just area name and domain.\nWhen a user asks to turn on all devices of a specific type, ask user to specify an area, unless there is only one device of that type.\nThis device is not able to start timers.\nAn overview of the areas and the devices in this smart home:\n- names: Zhuwo Light\n  domain: switch\n  state: 'off'\n- names: Ciwo Light\n  domain: switch\n  state: 'off'\n- names: Shufang Light\n  domain: switch\n  state: 'off'\n- names: Keting Light\n  domain: switch\n  state: 'off'\n- names: Weishengjian Light\n  domain: switch\n  state: 'off'\n- names: Guodao Light\n  domain: switch\n  state: 'off'\n- names: Chufang Light\n  domain: switch\n  state: 'off'\n"}, {'role': 'user', 'content': '客厅的光线有点暗'}]
2024-09-08 11:52:29.732 DEBUG (MainThread) [homeassistant.components.openai_conversation] Tools: [{'type': 'function', 'function': {'name': 'HassTurnOn', 'parameters': {'type': 'object', 'properties': {'name': {'type': 'string'}, 'area': {'type': 'string'}, 'floor': {'type': 'string'}, 'domain': {'type': 'array', 'items': {'type': 'string'}}, 'device_class': {'type': 'array', 'items': {'type': 'string', 'enum': ['outlet', 'switch', 'awning', 'blind', 'curtain', 'damper', 'door', 'garage', 'gate', 'shade', 'shutter', 'window', 'water', 'gas', 'tv', 'speaker', 'receiver']}}}, 'required': []}, 'description': 'Turns on/opens a device or entity'}}, {'type': 'function', 'function': {'name': 'HassTurnOff', 'parameters': {'type': 'object', 'properties': {'name': {'type': 'string'}, 'area': {'type': 'string'}, 'floor': {'type': 'string'}, 'domain': {'type': 'array', 'items': {'type': 'string'}}, 'device_class': {'type': 'array', 'items': {'type': 'string', 'enum': ['outlet', 'switch', 'awning', 'blind', 'curtain', 'damper', 'door', 'garage', 'gate', 'shade', 'shutter', 'window', 'water', 'gas', 'tv', 'speaker', 'receiver']}}}, 'required': []}, 'description': 'Turns off/closes a device or entity'}}]
```

prompt 以及用户输入的文本「客厅的光线有点暗」组成了消息列表，然后通过调用模型的方法来生成对话的回复，使用指定的模型、消息列表和其他配置参数，在调试日志中查看模型给出的响应，在 `tool_calls` 的响应中，包含了要执行的动作，调用 tool 方法：`HassTurnOn({'name': 'Keting Light', 'domain': 'switch'})`，调用 Home Assistant 服务，对客厅灯这个实体执行打开动作，然后返回输出消息「已经为您打开了客厅的灯。」

```
2024-09-08 11:52:32.505 DEBUG (MainThread) [homeassistant.components.openai_conversation] Response ChatCompletion(id='chatcmpl-307e0131-fb09-9e0b-8476-32d7eedfd59f', choices=[Choice(finish_reason='tool_calls', index=0, logprobs=None, message=ChatCompletionMessage(content='', role='assistant', function_call=None, tool_calls=[ChatCompletionMessageToolCall(id='call_079474c7a404411fa6518d', function=Function(arguments='{"name": "Keting Light", "domain": "switch"}', name='HassTurnOn'), type='function', index=0)]))], created=1725796356, model='qwen-plus', object='chat.completion', service_tier=None, system_fingerprint=None, usage=CompletionUsage(completion_tokens=26, prompt_tokens=741, total_tokens=767))
2024-09-08 11:52:32.505 DEBUG (MainThread) [homeassistant.components.openai_conversation] Tool call: HassTurnOn({'name': 'Keting Light', 'domain': 'switch'})
2024-09-08 11:52:32.510 DEBUG (MainThread) [homeassistant.components.openai_conversation] Tool response: {'speech': {}, 'response_type': 'action_done', 'data': {'targets': [], 'success': [{'name': 'Keting Light', 'type': <IntentResponseTargetType.ENTITY: 'entity'>, 'id': 'switch.keting_light'}], 'failed': []}}
2024-09-08 11:52:33.462 DEBUG (MainThread) [homeassistant.components.openai_conversation] Response ChatCompletion(id='chatcmpl-4afca46f-0ffa-98d0-be34-715d2c15f8b1', choices=[Choice(finish_reason='stop', index=0, logprobs=None, message=ChatCompletionMessage(content='已经为您打开了客厅的灯。', role='assistant', function_call=None, tool_calls=None))], created=1725796357, model='qwen-plus', object='chat.completion', service_tier=None, system_fingerprint=None, usage=CompletionUsage(completion_tokens=10, prompt_tokens=825, total_tokens=835))
```

接下来，在对话中继续输入「我想在客厅睡觉了」，在输出的 Prompt 里，除了新的语句，还包含了用户在这轮对话的上下文信息，包括之前 assisant 执行的动作：

```
2024-09-08 11:52:48.667 DEBUG (MainThread) [homeassistant.components.openai_conversation] Prompt: [{'role': 'system', 'content': "Current time is 19:52:48. Today's date is 2024-09-08.\nYou are a voice assistant for Home Assistant.\nAnswer questions about the world truthfully.\nAnswer in plain text. Keep it simple and to the point.\nWhen controlling Home Assistant always call the intent tools. Use HassTurnOn to lock and HassTurnOff to unlock a lock. When controlling a device, prefer passing just name and domain. When controlling an area, prefer passing just area name and domain.\nWhen a user asks to turn on all devices of a specific type, ask user to specify an area, unless there is only one device of that type.\nThis device is not able to start timers.\nAn overview of the areas and the devices in this smart home:\n- names: Zhuwo Light\n  domain: switch\n  state: 'off'\n- names: Ciwo Light\n  domain: switch\n  state: 'off'\n- names: Shufang Light\n  domain: switch\n  state: 'off'\n- names: Keting Light\n  domain: switch\n  state: 'on'\n- names: Weishengjian Light\n  domain: switch\n  state: 'off'\n- names: Guodao Light\n  domain: switch\n  state: 'off'\n- names: Chufang Light\n  domain: switch\n  state: 'off'\n"}, {'role': 'user', 'content': '客厅的光线有点暗'}, {'role': 'assistant', 'content': '', 'tool_calls': [{'id': 'call_079474c7a404411fa6518d', 'function': {'arguments': '{"name": "Keting Light", "domain": "switch"}', 'name': 'HassTurnOn'}, 'type': 'function'}]}, {'role': 'tool', 'tool_call_id': 'call_079474c7a404411fa6518d', 'content': '{"speech": {}, "response_type": "action_done", "data": {"targets": [], "success": [{"name": "Keting Light", "type": "entity", "id": "switch.keting_light"}], "failed": []}}'}, {'role': 'assistant', 'content': '已经为您打开了客厅的灯。'}, {'role': 'user', 'content': '我想在客厅睡觉了'}]
2024-09-08 11:52:48.668 DEBUG (MainThread) [homeassistant.components.openai_conversation] Tools: [{'type': 'function', 'function': {'name': 'HassTurnOn', 'parameters': {'type': 'object', 'properties': {'name': {'type': 'string'}, 'area': {'type': 'string'}, 'floor': {'type': 'string'}, 'domain': {'type': 'array', 'items': {'type': 'string'}}, 'device_class': {'type': 'array', 'items': {'type': 'string', 'enum': ['outlet', 'switch', 'awning', 'blind', 'curtain', 'damper', 'door', 'garage', 'gate', 'shade', 'shutter', 'window', 'water', 'gas', 'tv', 'speaker', 'receiver']}}}, 'required': []}, 'description': 'Turns on/opens a device or entity'}}, {'type': 'function', 'function': {'name': 'HassTurnOff', 'parameters': {'type': 'object', 'properties': {'name': {'type': 'string'}, 'area': {'type': 'string'}, 'floor': {'type': 'string'}, 'domain': {'type': 'array', 'items': {'type': 'string'}}, 'device_class': {'type': 'array', 'items': {'type': 'string', 'enum': ['outlet', 'switch', 'awning', 'blind', 'curtain', 'damper', 'door', 'garage', 'gate', 'shade', 'shutter', 'window', 'water', 'gas', 'tv', 'speaker', 'receiver']}}}, 'required': []}, 'description': 'Turns off/closes a device or entity'}}]
```

在响应中，显示调用 tool 方法：`HassTurnOff({'name': 'Keting Light', 'domain': 'switch'})`，关闭客厅的灯，然后返回输出消息「已经为您关闭了客厅的灯，祝您睡个好觉。」

```
2024-09-08 11:52:51.663 DEBUG (MainThread) [homeassistant.components.openai_conversation] Response ChatCompletion(id='chatcmpl-b5276269-533f-9576-9680-b69d640b1a67', choices=[Choice(finish_reason='tool_calls', index=0, logprobs=None, message=ChatCompletionMessage(content='', role='assistant', function_call=None, tool_calls=[ChatCompletionMessageToolCall(id='call_1ea61fbb79684e48baf6a0', function=Function(arguments='{"name": "Keting Light", "domain": "switch"}', name='HassTurnOff'), type='function', index=0)]))], created=1725796375, model='qwen-plus', object='chat.completion', service_tier=None, system_fingerprint=None, usage=CompletionUsage(completion_tokens=26, prompt_tokens=849, total_tokens=875))
2024-09-08 11:52:51.663 DEBUG (MainThread) [homeassistant.components.openai_conversation] Tool call: HassTurnOff({'name': 'Keting Light', 'domain': 'switch'})
2024-09-08 11:52:51.669 DEBUG (MainThread) [homeassistant.components.openai_conversation] Tool response: {'speech': {}, 'response_type': 'action_done', 'data': {'targets': [], 'success': [{'name': 'Keting Light', 'type': <IntentResponseTargetType.ENTITY: 'entity'>, 'id': 'switch.keting_light'}], 'failed': []}}
2024-09-08 11:52:52.813 DEBUG (MainThread) [homeassistant.components.openai_conversation] Response ChatCompletion(id='chatcmpl-04d55761-fc4d-9618-8844-9a8065f0f226', choices=[Choice(finish_reason='stop', index=0, logprobs=None, message=ChatCompletionMessage(content='已经为您关闭了客厅的灯，祝您睡个好觉。', role='assistant', function_call=None, tool_calls=None))], created=1725796377, model='qwen-plus', object='chat.completion', service_tier=None, system_fingerprint=None, usage=CompletionUsage(completion_tokens=17, prompt_tokens=933, total_tokens=950))
```

在 Home Assistant 项目中 OpenAI Conversation 的源码中，可以看到调用的完整代码过程：
1. 初始化上下文，包括语言、设备 ID、用户输入等信息。
2. 如果配置了 LLM API，获取对应实例，并准备工具列表。
3. 处理对话 ID，如果没有提供，生成一个新的 ULID，记录对话历史。
4. 生成提示文本（prompt），将系统和用户信息传递给 LLM 进行处理。
5. 调用LLM生成回复，处理工具调用（如设备操作）。
6. 保存对话历史，并返回处理结果给用户。

```python
async def async_process(
    self, user_input: conversation.ConversationInput
) -> conversation.ConversationResult:
    """Process a sentence."""
    options = self.entry.options
    intent_response = intent.IntentResponse(language=user_input.language)
    llm_api: llm.APIInstance | None = None
    tools: list[ChatCompletionToolParam] | None = None
    user_name: str | None = None
    llm_context = llm.LLMContext(
        platform=DOMAIN,
        context=user_input.context,
        user_prompt=user_input.text,
        language=user_input.language,
        assistant=conversation.DOMAIN,
        device_id=user_input.device_id,
    )

    if options.get(CONF_LLM_HASS_API):
        try:
            llm_api = await llm.async_get_api(
                self.hass,
                options[CONF_LLM_HASS_API],
                llm_context,
            )
        except HomeAssistantError as err:
            LOGGER.error("Error getting LLM API: %s", err)
            intent_response.async_set_error(
                intent.IntentResponseErrorCode.UNKNOWN,
                f"Error preparing LLM API: {err}",
            )
            return conversation.ConversationResult(
                response=intent_response, conversation_id=user_input.conversation_id
            )
        tools = [
            _format_tool(tool, llm_api.custom_serializer) for tool in llm_api.tools
        ]

    if user_input.conversation_id is None:
        conversation_id = ulid.ulid_now()
        messages = []

    elif user_input.conversation_id in self.history:
        conversation_id = user_input.conversation_id
        messages = self.history[conversation_id]

    else:
        # Conversation IDs are ULIDs. We generate a new one if not provided.
        # If an old OLID is passed in, we will generate a new one to indicate
        # a new conversation was started. If the user picks their own, they
        # want to track a conversation and we respect it.
        try:
            ulid.ulid_to_bytes(user_input.conversation_id)
            conversation_id = ulid.ulid_now()
        except ValueError:
            conversation_id = user_input.conversation_id

        messages = []

    if (
        user_input.context
        and user_input.context.user_id
        and (
            user := await self.hass.auth.async_get_user(user_input.context.user_id)
        )
    ):
        user_name = user.name

    try:
        prompt_parts = [
            template.Template(
                llm.BASE_PROMPT
                + options.get(CONF_PROMPT, llm.DEFAULT_INSTRUCTIONS_PROMPT),
                self.hass,
            ).async_render(
                {
                    "ha_name": self.hass.config.location_name,
                    "user_name": user_name,
                    "llm_context": llm_context,
                },
                parse_result=False,
            )
        ]

    except TemplateError as err:
        LOGGER.error("Error rendering prompt: %s", err)
        intent_response = intent.IntentResponse(language=user_input.language)
        intent_response.async_set_error(
            intent.IntentResponseErrorCode.UNKNOWN,
            f"Sorry, I had a problem with my template: {err}",
        )
        return conversation.ConversationResult(
            response=intent_response, conversation_id=conversation_id
        )

    if llm_api:
        prompt_parts.append(llm_api.api_prompt)

    prompt = "\n".join(prompt_parts)

    # Create a copy of the variable because we attach it to the trace
    messages = [
        ChatCompletionSystemMessageParam(role="system", content=prompt),
        *messages[1:],
        ChatCompletionUserMessageParam(role="user", content=user_input.text),
    ]

    LOGGER.debug("Prompt: %s", messages)
    LOGGER.debug("Tools: %s", tools)
    trace.async_conversation_trace_append(
        trace.ConversationTraceEventType.AGENT_DETAIL,
        {"messages": messages, "tools": llm_api.tools if llm_api else None},
    )

    client = self.entry.runtime_data

    # To prevent infinite loops, we limit the number of iterations
    for _iteration in range(MAX_TOOL_ITERATIONS):
        try:
            result = await client.chat.completions.create(
                model=options.get(CONF_CHAT_MODEL, RECOMMENDED_CHAT_MODEL),
                messages=messages,
                tools=tools or NOT_GIVEN,
                max_tokens=options.get(CONF_MAX_TOKENS, RECOMMENDED_MAX_TOKENS),
                top_p=options.get(CONF_TOP_P, RECOMMENDED_TOP_P),
                temperature=options.get(CONF_TEMPERATURE, RECOMMENDED_TEMPERATURE),
                user=conversation_id,
            )
        except openai.OpenAIError as err:
            intent_response = intent.IntentResponse(language=user_input.language)
            intent_response.async_set_error(
                intent.IntentResponseErrorCode.UNKNOWN,
                f"Sorry, I had a problem talking to OpenAI: {err}",
            )
            return conversation.ConversationResult(
                response=intent_response, conversation_id=conversation_id
            )

        LOGGER.debug("Response %s", result)
        response = result.choices[0].message

        def message_convert(
            message: ChatCompletionMessage,
        ) -> ChatCompletionMessageParam:
            """Convert from class to TypedDict."""
            tool_calls: list[ChatCompletionMessageToolCallParam] = []
            if message.tool_calls:
                tool_calls = [
                    ChatCompletionMessageToolCallParam(
                        id=tool_call.id,
                        function=Function(
                            arguments=tool_call.function.arguments,
                            name=tool_call.function.name,
                        ),
                        type=tool_call.type,
                    )
                    for tool_call in message.tool_calls
                ]
            param = ChatCompletionAssistantMessageParam(
                role=message.role,
                content=message.content,
            )
            if tool_calls:
                param["tool_calls"] = tool_calls
            return param

        messages.append(message_convert(response))
        tool_calls = response.tool_calls

        if not tool_calls or not llm_api:
            break

        for tool_call in tool_calls:
            tool_input = llm.ToolInput(
                tool_name=tool_call.function.name,
                tool_args=json.loads(tool_call.function.arguments),
            )
            LOGGER.debug(
                "Tool call: %s(%s)", tool_input.tool_name, tool_input.tool_args
            )

            try:
                tool_response = await llm_api.async_call_tool(tool_input)
            except (HomeAssistantError, vol.Invalid) as e:
                tool_response = {"error": type(e).__name__}
                if str(e):
                    tool_response["error_text"] = str(e)

            LOGGER.debug("Tool response: %s", tool_response)
            messages.append(
                ChatCompletionToolMessageParam(
                    role="tool",
                    tool_call_id=tool_call.id,
                    content=json.dumps(tool_response),
                )
            )

    self.history[conversation_id] = messages

    intent_response = intent.IntentResponse(language=user_input.language)
    intent_response.async_set_speech(response.content or "")
    return conversation.ConversationResult(
        response=intent_response, conversation_id=conversation_id
    )
```
