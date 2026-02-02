import logging
import asyncio
from pydantic import Field
from typing import Annotated
from dotenv import load_dotenv


from livekit import agents, rtc
from livekit.agents import (
    AgentServer,
    AgentSession,
    Agent,
    inference,
    room_io,
    RunContext,
    function_tool,
)
from livekit.plugins import google, noise_cancellation, silero
from livekit.plugins.turn_detector.multilingual import MultilingualModel

load_dotenv(".env.local")

# logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)


class Assistant(Agent):
    def __init__(self) -> None:
        super().__init__(
            instructions=(
                "You are a helpful, friendly voice AI assistant. "
                "You speak naturally and keep responses concise."
            )
        )

    @function_tool
    async def end_conversation(
        self,
        context: RunContext,
        reason: Annotated[str, Field(description="Why the call is ending")],
    ) -> None:
        logger.info(f"End requested: {reason}")

        # Just close the session - LLM already spoke the goodbye
        context.session.shutdown(drain=True)


server = AgentServer()


@server.rtc_session()
async def entrypoint(ctx: agents.JobContext):
    session = AgentSession(
        llm=google.realtime.RealtimeModel(
            model="gemini-2.5-flash-native-audio-preview-12-2025",
            voice="Puck",
            instructions=(
                "You are a helpful, friendly voice AI assistant with a warm and engaging personality. "
                "You assist users with their questions and requests using your extensive knowledge. "
                "Keep your responses concise, natural, and conversational - like speaking to a friend. "
                "Avoid complex formatting, emojis, or special punctuation since this is a voice interaction. "
                "Be curious, show genuine interest in the user's questions, and add a touch of humor when appropriate. "
                "If the user asks to end the call, says goodbye, or wants to finish the conversation, "
                "you MUST ALWAYS say a short, polite goodbye message FIRST, then and ONLY then call the end_conversation tool. "
                "Do NOT call the tool before speaking your goodbye. "
                "The goodbye must be spoken as part of your response before any tool calls."
            ),
        ),
        vad=silero.VAD.load(),
        turn_detection=MultilingualModel(),
        userdata={},
    )

    @session.on("conversation_item_added")
    def on_conversation_item_added(ev):
        if hasattr(ev, "item"):
            logger.info(f"[Chat] {ev.item.role}: {ev.item.content}")

    await session.start(
        room=ctx.room,
        agent=Assistant(),
        room_options=room_io.RoomOptions(
            delete_room_on_close=True,
            audio_input=room_io.AudioInputOptions(
                noise_cancellation=lambda params: (
                    noise_cancellation.BVCTelephony()
                    if params.participant.kind
                    == rtc.ParticipantKind.PARTICIPANT_KIND_SIP
                    else noise_cancellation.BVC()
                ),
            ),
        ),
    )

    await session.generate_reply(
        instructions="Greet the user warmly and ask how you can help.",
    )


if __name__ == "__main__":
    agents.cli.run_app(server)
