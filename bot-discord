import discord
from discord.ext import commands
import openai
import asyncio
import os

# CONFIG (SEGURA)
TOKEN = os.getenv("TOKEN")
OPENAI_KEY = os.getenv("OPENAI_KEY")
CANAL_DENUNCIA_ID = 1234567890  # COLOCA O ID DO CANAL

# CARGOS PERMITIDOS
CARGOS_AUTORIZADOS = ["👑DONO", "🪬BOT-CYBER", "💻ADM"]

# IA
openai.api_key = OPENAI_KEY

intents = discord.Intents.all()
bot = commands.Bot(command_prefix="!", intents=intents)

print("🔥 Iniciando HIPER BOT...")

@bot.event
async def on_ready():
    print(f"🔥 HIPER BOT ONLINE: {bot.user}")

# IA (30 segundos)
async def analisar_ia(texto):
    try:
        resposta = await asyncio.wait_for(
            openai.ChatCompletion.acreate(
                model="gpt-4o-mini",
                messages=[
                    {"role": "system", "content": "Você é uma IA de moderação. Analise a mensagem e diga risco de 0 a 10 e tipo (spam, ofensa ou normal)."},
                    {"role": "user", "content": texto}
                ]
            ),
            timeout=30
        )

        return resposta.choices[0].message.content

    except Exception as e:
        return f"Erro IA: {e}"

# PERMISSÃO
def tem_permissao(member):
    for cargo in member.roles:
        if cargo.name in CARGOS_AUTORIZADOS:
            return True
    return False

@bot.event
async def on_message(message):
    if message.author.bot:
        return

    texto = message.content.lower()

    # BLOQUEAR LINKS
    if "http" in texto:
        if not tem_permissao(message.author):
            await message.delete()
            await message.channel.send(f"🚫 {message.author.mention} link não permitido!")
            return

    # IA
    analise = await analisar_ia(texto)

    # DETECÇÃO EXTRA
    risco = 0
    tipo = "normal"

    palavras_ruins = ["lixo", "fdp", "vai se lascar", "vai tomar"]
    for p in palavras_ruins:
        if p in texto:
            risco += 3
            tipo = "ofensa"

    if "http" in texto:
        risco += 2
        tipo = "link"

    if risco >= 3:
        canal = bot.get_channel(CANAL_DENUNCIA_ID)

        embed = discord.Embed(
            title="🚨 HIPER DETECÇÃO",
            color=discord.Color.red()
        )

        embed.add_field(name="Usuário", value=message.author.mention, inline=False)
        embed.add_field(name="Mensagem", value=message.content, inline=False)
        embed.add_field(name="IA", value=analise, inline=False)
        embed.add_field(name="Tipo", value=tipo, inline=True)
        embed.add_field(name="Risco", value=risco, inline=True)

        await canal.send(embed=embed)

    await bot.process_commands(message)

bot.run(TOKEN)
