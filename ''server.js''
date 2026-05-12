const express = require('express');
const path = require('path');
const crypto = require('crypto');
const { Telegraf } = require('telegraf');

const BOT_TOKEN = '8689775862:AAGZYQsoABr76IQgJCRnL-RiFQ6Ws8cRDog';
const PORT = process.env.PORT || 3000;
const WEB_APP_URL = process.env.RENDER_EXTERNAL_URL || 'http://localhost:3000';

console.log('WEB_APP_URL:', WEB_APP_URL);

const app = express();
const bot = new Telegraf(BOT_TOKEN);

app.use(express.json());
app.use(express.static(__dirname));

const products = [
  { id: 1, name: 'Basic', price: 990, period: 'мес', emoji: '🌟' },
  { id: 2, name: 'Pro', price: 2490, period: 'мес', emoji: '⚡' },
  { id: 3, name: 'Premium', price: 4990, period: 'мес', emoji: '💎' },
];

const orders = [];

app.get('/api/products', (req, res) => {
  res.json(products);
});

app.post('/api/pay', (req, res) => {
  const { productId, cardNumber, cardExpiry, cardCvc, cardHolder, email, telegramId } = req.body;

  const errors = [];
  const product = products.find(p => p.id === productId);

  if (!product) errors.push('Товар не найден');
  if (!cardNumber || !/^\d{16}$/.test(cardNumber.replace(/\s/g, ''))) errors.push('Номер карты: 16 цифр');
  if (!cardExpiry || !/^(0[1-9]|1[0-2])\/\d{2}$/.test(cardExpiry)) errors.push('Срок действия: ММ/ГГ');
  if (!cardCvc || !/^\d{3,4}$/.test(cardCvc)) errors.push('CVC: 3-4 цифры');
  if (!cardHolder || cardHolder.trim().length < 3) errors.push('Имя держателя обязательно');
  if (!email || !email.includes('@')) errors.push('Некорректный email');

  if (errors.length > 0) {
    return res.status(400).json({ success: false, errors });
  }

  const transactionId = 'txn_' + crypto.randomBytes(6).toString('hex');
  const maskedCard = '**** ' + cardNumber.replace(/\s/g, '').slice(-4);

  const order = {
    id: orders.length + 1,
    transactionId,
    product: product.name,
    amount: product.price,
    currency: 'RUB',
    cardMasked: maskedCard,
    cardHolder,
    email,
    telegramId: telegramId || null,
    status: 'paid',
    createdAt: new Date().toISOString(),
  };

  orders.push(order);

  setTimeout(() => {
    res.json({
      success: true,
      transactionId,
      amount: product.price,
      productName: product.name,
      maskedCard,
      message: `✅ Оплата ${product.price} ₽ прошла успешно!`,
    });

    if (telegramId) {
      bot.telegram.sendMessage(
        telegramId,
        `✅ *Оплата прошла успешно!*\n\n📦 Тариф: ${product.emoji} ${product.name}\n💰 Сумма: ${product.price} ₽\n💳 Карта: ${maskedCard}\n🆔 Транзакция: \`${transactionId}\`\n\nСпасибо за покупку!`,
        { parse_mode: 'Markdown' }
      ).catch(() => {});
    }
  }, 1500);
});

bot.start(async (ctx) => {
  const telegramId = ctx.from.id;
  const firstName = ctx.from.first_name || 'Пользователь';

  await ctx.reply(
    `👋 Привет, *${firstName}*!\n\nДобро пожаловать в магазин подписок.\nВыбери тариф и оплати прямо в Telegram.`,
    {
      parse_mode: 'Markdown',
      reply_markup: {
        inline_keyboard: [
          [{ text: '💳 Открыть форму оплаты', web_app: { url: WEB_APP_URL } }],
          [{ text: '📋 О боте', callback_data: 'about' }],
        ],
      },
    }
  );
});

bot.action('about', async (ctx) => {
  await ctx.answerCbQuery();
  await ctx.reply(
    `🤖 *Payment Bot*\n\nПринимает оплату через Telegram Web App.\nДанные карт не сохраняются.\n\nТарифы:\n` +
    products.map(p => `${p.emoji} *${p.name}* — ${p.price} ₽/мес`).join('\n'),
    { parse_mode: 'Markdown' }
  );
});

bot.command('help', (ctx) => {
  ctx.reply('Нажми /start чтобы открыть меню оплаты.');
});

app.listen(PORT, () => {
  console.log(`✅ Сервер: http://localhost:${PORT}`);
});

bot.launch(() => {
  console.log('🤖 Бот запущен');
});

process.once('SIGINT', () => bot.stop('SIGINT'));
process.once('SIGTERM', () => bot.stop('SIGTERM'));