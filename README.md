# virtual-ai

Features
Plug-and-Play Integration with VIRTUAL: Integrate conversational AI 3D models into your React applications by utilizing the customizable components.

Customizable Services and Hooks: To implement own components, you can use the useVirtual hook and other helper functions and classes to build your own components that integrate with VIRTUAL.

Usage
To install @virtual-protocol/react-virtual-ai in your React project, follow these simple steps:

Step 1: Installation
npm install @virtual-protocol/react-virtual-ai --save
or

yarn add @virtual-protocol/react-virtual-ai
Reference: https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-npm-registry

Step 2: Obtain Your API Key and Secret
Follow the VIRTUAL documentation on creating API key and secret.

Step 3: Implement initAccessToken function
initAccessToken prop is available for CharacterRoom component and useVirtual hook. The access token returned via this function will be used as the authorization of the runner service.

IMPORTANT NOTE: The UNSAFE_initAccessToken function is provided as an example implementation, please do not use it in production as exposing API key and secret is unsafe. To enable UNSAFE_initAccessToken, pass in metadata props to the CharacterRoom or useVirtual hook.

Sample metadata payload when using UNSAFE_initAccessToken function:

metadata={{
  apiKey: "<YOUR VIRTUAL API KEY>",
  apiSecret: "<YOUR VIRTUAL API SECRET>",
  userUid: "1", // any unique user identifier that will be used for runner memory core to remember conversations
  userName: "User", // user's name that the Virtual will address as
  }}
Recommended implementation of initAccessToken function:

// This function fetches runner access token and keep in cache
export const initAccessToken = async (
  virtualId: number,
  metadata?: { [id: string]: any }
) => {
  if (!virtualId) return "";
  // runner token by bot is saved as runnerToken<virtualId> in localStorage
  let cachedRunnerToken = localStorage.getItem(`runnerToken${virtualId}`) ?? "";
  // Fetch a new runner token if expired
  if (!cachedRunnerToken || !validateJwt(cachedRunnerToken)) {
    // Get runner token via your own dapp server
    const accessToken = localStorage.getItem("accessToken");
    if (!accessToken) return "";
    const resp = await fetch(`${YOUR_BACKEND_URL}/api/auth/token`, {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
        Authorization: `Bearer ${accessToken}`,
      },
      body: JSON.stringify({
        virtualId: virtualId,
        metadata: metadata,
      }),
    });
    const respJson = await resp.json();
    cachedRunnerToken = respJson.runnerToken ?? "";
    localStorage.setItem(`runnerToken${virtualId}`, cachedRunnerToken);
  }
  return cachedRunnerToken;
};
Step 4: Insert the CharacterRoom component
There

import {
  CharacterRoom,
  UNSAFE_initAccessToken,
} from "@virtual-protocol/react-virtual-ai";

return (
  <CharacterRoom
    userName="User"
    virtualName="Virtual Name"
    virtualId={1} // unique virtual id in number / string that will define the
    initAccessToken={UNSAFE_initAccessToken}
  />
);
VirtualService
The VirtualService class contains all operations that communicate with the Virtual runner service. It is also used in the useVirtual hook to get the prompt responses.

const virtualService = new VirtualService({
  virtualId: 1,
  userName: "",
  virtualName: "",
  initAccessToken: undefined,
  onPromptError: undefined,
  metadata: undefined,
});
Other Usages
There are also other services and functions that you can explore to customize your own components

useVirtual hook
This hook is a wrapper to the VirtualService class that allows you to communicate with Virtual with the functions returned such as createPrompt and virtualService.getTTSResponse.

const { modelUrl, createPrompt, virtualService } = useVirtual({
  virtualId: 1,
  userName: "",
  virtualName: "",
  initAccessToken: undefined,
  onPromptError: undefined,
  metadata: undefined,
});
Utils
There are several util functions provided, you may utilize separately for specific cases.

UNSAFE_initAccessToken: Default function to generate access token. DO NOT use this in production.

validateJwt: Validate JWT token expiry

getVirtualRunnerUrl: Get virtual runner URL via JWT token

getQuotedTexts: Get quoted texts as list via string input

loadAnimation: Load VMD / Mixamo FBX animation into VRM model

fadeByEmotion: Fade in facial expression for the VRM model

blink: Blink the VRM model's eyes

Components
AICharacter: 3D Character with PresentationControl. Must wrap inside ThreeJS Canvas. Utilized in CharacterScene.

CharacterScene: 3D Character with Scene. May use this for displaying 3D model and animations. Utilized in CharacterRoom.

CharacterInput: Input component for prompting. Utilized in CharacterRoom.

CharacterRoom: Full component for React Virtual AI.

Services
VirtualService: VirtualService contains all operations required to send prompt to the VIRTUAL.

VrmService: VrmService provides functions to load VRM model and fade to animations.

API References
JSDocs comments are available in the components, classes and functions. Kindly follow the comments for more details.
