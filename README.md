# STOCK_PRICE_PREDICTION
An interactive Stock Price Prediction Dashboard built using Machine Learning and Python that analyzes historical stock data to forecast future trends. This project provides users with visual insights through graphs and an intuitive web interface, helping in better understanding of stock market behavior.
OPEN API
openapi: 3.1.0
info:
  title: Api
  version: 0.1.0
  description: API specification
servers:
  - url: /api
tags:
  - name: health
  - name: stocks
paths:
  /healthz:
    get:
      operationId: healthCheck
      tags: [health]
      responses:
        "200":
          description: Healthy
          content:
            application/json:
              schema: { $ref: "#/components/schemas/HealthStatus" }

  /stocks/popular:
    get:
      operationId: getPopularTickers
      tags: [stocks]
      summary: Popular ticker watchlist
      responses:
        "200":
          description: Popular tickers
          content:
            application/json:
              schema: { $ref: "#/components/schemas/PopularTickersResponse" }

  /stocks/{ticker}/quote:
    get:
      operationId: getStockQuote
      tags: [stocks]
      parameters:
        - { name: ticker, in: path, required: true, schema: { type: string } }
      responses:
        "200":
          description: Current quote
          content:
            application/json:
              schema: { $ref: "#/components/schemas/StockQuote" }
        "404":
          description: Ticker not found
          content:
            application/json:
              schema: { $ref: "#/components/schemas/ErrorResponse" }

  /stocks/{ticker}/history:
    get:
      operationId: getStockHistory
      tags: [stocks]
      parameters:
        - { name: ticker,    in: path,  required: true,  schema: { type: string } }
        - { name: startDate, in: query, required: false, schema: { type: string, format: date } }
        - { name: endDate,   in: query, required: false, schema: { type: string, format: date } }
      responses:
        "200":
          description: Historical stock data
          content:
            application/json:
              schema: { $ref: "#/components/schemas/StockHistoryResponse" }
        "404":
          description: Ticker not found
          content:
            application/json:
              schema: { $ref: "#/components/schemas/ErrorResponse" }

  /stocks/{ticker}/predict:
    get:
      operationId: predictStockPrice
      tags: [stocks]
      summary: Predict future stock prices using LSTM
      parameters:
        - { name: ticker,    in: path,  required: true,  schema: { type: string } }
        - { name: startDate, in: query, required: false, schema: { type: string, format: date } }
        - { name: endDate,   in: query, required: false, schema: { type: string, format: date } }
        - { name: horizon,   in: query, required: false, schema: { type: integer, minimum: 1, maximum: 60, default: 14 } }
      responses:
        "200":
          description: Prediction result
          content:
            application/json:
              schema: { $ref: "#/components/schemas/PredictionResponse" }
        "404":
          description: Ticker not found
          content:
            application/json:
              schema: { $ref: "#/components/schemas/ErrorResponse" }

components:
  schemas:
    HealthStatus:
      type: object
      required: [status]
      properties:
        status: { type: string }

    ErrorResponse:
      type: object
      required: [error, message]
      properties:
        error:   { type: string }
        message: { type: string }

    StockPricePoint:
      type: object
      required: [date, open, high, low, close, adjClose, volume]
      properties:
        date:     { type: string, format: date }
        open:     { type: number }
        high:     { type: number }
        low:      { type: number }
        close:    { type: number }
        adjClose: { type: number }
        volume:   { type: number }

    StockHistorySummary:
      type: object
      required: [startPrice, endPrice, highPrice, lowPrice, averageVolume, totalReturnPct, volatilityPct]
      properties:
        startPrice:     { type: number }
        endPrice:       { type: number }
        highPrice:      { type: number }
        lowPrice:       { type: number }
        averageVolume:  { type: number }
        totalReturnPct: { type: number }
        volatilityPct:  { type: number }

    StockHistoryResponse:
      type: object
      required: [ticker, currency, startDate, endDate, prices, summary]
      properties:
        ticker:    { type: string }
        currency:  { type: string }
        startDate: { type: string, format: date }
        endDate:   { type: string, format: date }
        prices:    { type: array, items: { $ref: "#/components/schemas/StockPricePoint" } }
        summary:   { $ref: "#/components/schemas/StockHistorySummary" }

    StockQuote:
      type: object
      required:
        [ticker, shortName, currency, regularMarketPrice, regularMarketChange,
         regularMarketChangePercent, regularMarketDayHigh, regularMarketDayLow,
         regularMarketVolume, fiftyTwoWeekHigh, fiftyTwoWeekLow, previousClose]
      properties:
        ticker:                     { type: string }
        shortName:                  { type: string }
        longName:                   { type: string }
        currency:                   { type: string }
        exchange:                   { type: string }
        regularMarketPrice:         { type: number }
        regularMarketChange:        { type: number }
        regularMarketChangePercent: { type: number }
        regularMarketDayHigh:       { type: number }
        regularMarketDayLow:        { type: number }
        regularMarketVolume:        { type: number }
        marketCap:                  { type: [number, "null"] }
        fiftyTwoWeekHigh:           { type: number }
        fiftyTwoWeekLow:            { type: number }
        previousClose:              { type: number }

    PredictedPoint:
      type: object
      required: [date, predictedClose]
      properties:
        date:           { type: string, format: date }
        predictedClose: { type: number }

    BacktestPoint:
      type: object
      required: [date, actualClose, predictedClose]
      properties:
        date:           { type: string, format: date }
        actualClose:    { type: number }
        predictedClose: { type: number }

    PredictionMetrics:
      type: object
      required: [rmse, mae, mape, directionalAccuracy, trainSamples, testSamples, epochs, windowSize]
      properties:
        rmse:                { type: number }
        mae:                 { type: number }
        mape:                { type: number }
        directionalAccuracy: { type: number }
        trainSamples:        { type: integer }
        testSamples:         { type: integer }
        epochs:              { type: integer }
        windowSize:          { type: integer }

    PredictionResponse:
      type: object
      required: [ticker, currency, modelName, horizon, forecast, backtest, metrics, lastActualClose, lastActualDate]
      properties:
        ticker:          { type: string }
        currency:        { type: string }
        modelName:       { type: string }
        horizon:         { type: integer }
        forecast:        { type: array, items: { $ref: "#/components/schemas/PredictedPoint" } }
        backtest:        { type: array, items: { $ref: "#/components/schemas/BacktestPoint" } }
        metrics:         { $ref: "#/components/schemas/PredictionMetrics" }
        lastActualClose: { type: number }
        lastActualDate:  { type: string, format: date }

    PopularTicker:
      type: object
      required: [ticker, name, price, changePercent]
      properties:
        ticker:        { type: string }
        name:          { type: string }
        price:         { type: number }
        changePercent: { type: number }

    PopularTickersResponse:
      type: object
      required: [tickers]
      properties:
        tickers: { type: array, items: { $ref: "#/components/schemas/PopularTicker" } }

BACKEND
{
  "name": "@workspace/api-server",
  "type": "module",
  "scripts": {
    "dev":   "export NODE_ENV=development && pnpm run build && pnpm run start",
    "build": "node ./build.mjs",
    "start": "node --enable-source-maps ./dist/index.mjs"
  },
  "dependencies": {
    "@tensorflow/tfjs": "^4.22.0",
    "@workspace/api-zod": "workspace:*",
    "cookie-parser": "^1.4.7",
    "cors": "^2",
    "express": "^5",
    "long": "^5.3.2",
    "pino": "^9",
    "pino-http": "^10",
    "yahoo-finance2": "^3.14.0",
    "zod": "catalog:"
  }
}
import YahooFinance from "yahoo-finance2";
import { logger } from "./logger";

// yahoo-finance2 v3 — default export is a CLASS, must be `new`'d
const yahooFinance = new (YahooFinance as any)();
(yahooFinance as any).suppressNotices?.(["yahooSurvey"]);

export interface PricePoint {
  date: string; open: number; high: number; low: number;
  close: number; adjClose: number; volume: number;
}

export interface QuoteSummary {
  ticker: string; shortName: string; longName?: string;
  currency: string; exchange?: string;
  regularMarketPrice: number; regularMarketChange: number; regularMarketChangePercent: number;
  regularMarketDayHigh: number; regularMarketDayLow: number; regularMarketVolume: number;
  marketCap: number | null;
  fiftyTwoWeekHigh: number; fiftyTwoWeekLow: number;
  previousClose: number;
}

const TWO_YEARS_MS = 1000 * 60 * 60 * 24 * 365 * 2;
const toISODate = (d: Date) => d.toISOString().slice(0, 10);

export function defaultDateRange(start?: Date, end?: Date) {
  const endDate = end ?? new Date();
  const startDate = start ?? new Date(endDate.getTime() - TWO_YEARS_MS);
  return { startDate, endDate };
}

export async function fetchHistory(ticker: string, start: Date, end: Date): Promise<PricePoint[]> {
  const upper = ticker.toUpperCase().trim();
  const result = (await yahooFinance.chart(upper, {
    period1: start, period2: end, interval: "1d",
  })) as any;

  const quotes = (result?.quotes ?? []) as Array<any>;
  const points: PricePoint[] = [];
  for (const q of quotes) {
    if (!q.date || q.open == null || q.high == null || q.low == null || q.close == null || q.volume == null) continue;
    points.push({
      date: toISODate(new Date(q.date)),
      open: +q.open, high: +q.high, low: +q.low, close: +q.close,
      adjClose: +(q.adjclose ?? q.close), volume: +q.volume,
    });
  }
  return points;
}

export async function fetchQuote(ticker: string): Promise<QuoteSummary> {
  const upper = ticker.toUpperCase().trim();
  const q = (await yahooFinance.quote(upper)) as Record<string, unknown> | null;
  if (!q || q["regularMarketPrice"] == null) throw new Error(`No quote for ${upper}`);

  const num = (k: string, fb = 0) => {
    const v = q[k]; return v == null || Number.isNaN(Number(v)) ? fb : Number(v);
  };
  const str = (k: string) => (typeof q[k] === "string" ? (q[k] as string) : undefined);
  const price = num("regularMarketPrice");
  const mc = q["marketCap"];

  return {
    ticker: upper,
    shortName: str("shortName") ?? upper,
    longName: str("longName"),
    currency: str("currency") ?? "USD",
    exchange: str("fullExchangeName") ?? str("exchange"),
    regularMarketPrice: price,
    regularMarketChange: num("regularMarketChange"),
    regularMarketChangePercent: num("regularMarketChangePercent"),
    regularMarketDayHigh: num("regularMarketDayHigh", price),
    regularMarketDayLow:  num("regularMarketDayLow",  price),
    regularMarketVolume:  num("regularMarketVolume"),
    marketCap: mc != null && !Number.isNaN(Number(mc)) ? Number(mc) : null,
    fiftyTwoWeekHigh: num("fiftyTwoWeekHigh", price),
    fiftyTwoWeekLow:  num("fiftyTwoWeekLow",  price),
    previousClose:    num("regularMarketPreviousClose", price),
  };
}

export function summarize(prices: PricePoint[]) {
  if (!prices.length) return { startPrice:0, endPrice:0, highPrice:0, lowPrice:0, averageVolume:0, totalReturnPct:0, volatilityPct:0 };
  const startPrice = prices[0]!.close;
  const endPrice   = prices[prices.length - 1]!.close;
  let high = -Infinity, low = Infinity, volSum = 0;
  for (const p of prices) { if (p.high > high) high = p.high; if (p.low < low) low = p.low; volSum += p.volume; }
  const returns: number[] = [];
  for (let i = 1; i < prices.length; i++) {
    const prev = prices[i - 1]!.close, curr = prices[i]!.close;
    if (prev > 0) returns.push(Math.log(curr / prev));
  }
  const mean = returns.reduce((a, b) => a + b, 0) / Math.max(returns.length, 1);
  const variance = returns.reduce((a, b) => a + (b - mean) ** 2, 0) / Math.max(returns.length - 1, 1);
  const annualVolPct = Math.sqrt(variance) * Math.sqrt(252) * 100;
  return {
    startPrice, endPrice, highPrice: high, lowPrice: low,
    averageVolume: volSum / prices.length,
    totalReturnPct: ((endPrice - startPrice) / startPrice) * 100,
    volatilityPct: annualVolPct,
  };
}

export function nextBusinessDays(fromDate: Date, count: number): string[] {
  const out: string[] = []; const d = new Date(fromDate);
  while (out.length < count) {
    d.setDate(d.getDate() + 1);
    const day = d.getUTCDay();
    if (day === 0 || day === 6) continue;
    out.push(toISODate(d));
  }
  return out;
}

export function logYahooError(err: unknown, ticker: string) {
  logger.error({ err, ticker }, "Yahoo Finance error");
}
import * as tf from "@tensorflow/tfjs";
import type { PricePoint } from "./stockData";
import { nextBusinessDays } from "./stockData";

export interface PredictionMetrics {
  rmse: number; mae: number; mape: number; directionalAccuracy: number;
  trainSamples: number; testSamples: number; epochs: number; windowSize: number;
}
export interface BacktestPoint { date: string; actualClose: number; predictedClose: number; }
export interface ForecastPoint { date: string; predictedClose: number; }
export interface PredictionResult {
  modelName: string; forecast: ForecastPoint[]; backtest: BacktestPoint[];
  metrics: PredictionMetrics; lastActualClose: number; lastActualDate: string;
}

interface BuildOptions { windowSize: number; epochs: number; testSplit: number; horizon: number; }
const DEFAULT_OPTIONS: BuildOptions = { windowSize: 30, epochs: 8, testSplit: 0.15, horizon: 14 };

function normalize(values: number[]) {
  let min = Infinity, max = -Infinity;
  for (const v of values) { if (v < min) min = v; if (v > max) max = v; }
  const range = max - min || 1;
  return { scaled: values.map((v) => (v - min) / range), min, max };
}
const denormalize = (s: number, min: number, max: number) => s * ((max - min) || 1) + min;

function makeWindows(series: number[], w: number) {
  const xs: number[][] = [], ys: number[] = [];
  for (let i = 0; i + w < series.length; i++) { xs.push(series.slice(i, i + w)); ys.push(series[i + w]!); }
  return { xs, ys };
}

function buildModel(windowSize: number): tf.Sequential {
  const model = tf.sequential();
  model.add(tf.layers.lstm({ units: 32, inputShape: [windowSize, 1], returnSequences: false }));
  model.add(tf.layers.dropout({ rate: 0.1 }));
  model.add(tf.layers.dense({ units: 1 }));
  model.compile({ optimizer: tf.train.adam(0.01), loss: "meanSquaredError" });
  return model;
}

export async function predictPrices(prices: PricePoint[], opts: Partial<BuildOptions> = {}): Promise<PredictionResult> {
  const { windowSize, epochs, testSplit, horizon } = { ...DEFAULT_OPTIONS, ...opts };
  if (prices.length < windowSize + 30) {
    throw new Error(`Not enough data to train (have ${prices.length}, need at least ${windowSize + 30}).`);
  }

  const closes = prices.map((p) => p.close);
  const { scaled, min, max } = normalize(closes);

  const { xs, ys } = makeWindows(scaled, windowSize);
  const total = xs.length;
  const testCount = Math.max(Math.min(60, Math.floor(total * testSplit)), 20);
  const trainCount = total - testCount;

  const trainXs = xs.slice(0, trainCount), trainYs = ys.slice(0, trainCount);
  const testXs  = xs.slice(trainCount),    testYs  = ys.slice(trainCount);

  const trainXTensor = tf.tensor3d(trainXs.map(w => w.map(v => [v])), [trainXs.length, windowSize, 1]);
  const trainYTensor = tf.tensor2d(trainYs.map(v => [v]), [trainYs.length, 1]);

  const model = buildModel(windowSize);
  await model.fit(trainXTensor, trainYTensor, { epochs, batchSize: 32, shuffle: true, verbose: 0 });

  // Backtest
  const testXTensor = tf.tensor3d(testXs.map(w => w.map(v => [v])), [testXs.length, windowSize, 1]);
  const testPredsTensor = model.predict(testXTensor) as tf.Tensor;
  const testPredsScaled = Array.from(await testPredsTensor.data());
  testPredsTensor.dispose();

  const backtest: BacktestPoint[] = [];
  let sqErr = 0, absErr = 0, pctErr = 0, dirHits = 0, dirTotal = 0;

  for (let i = 0; i < testYs.length; i++) {
    const idx = trainCount + i + windowSize;
    const actual = closes[idx]!;
    const predicted = denormalize(testPredsScaled[i]!, min, max);
    backtest.push({ date: prices[idx]!.date, actualClose: actual, predictedClose: predicted });

    sqErr  += (actual - predicted) ** 2;
    absErr += Math.abs(actual - predicted);
    pctErr += Math.abs((actual - predicted) / actual);

    if (i > 0) {
      const prevActual = closes[idx - 1]!;
      const aDir = Math.sign(actual - prevActual);
      const pDir = Math.sign(predicted - prevActual);
      if (aDir !== 0 && aDir === pDir) dirHits++;
      if (aDir !== 0) dirTotal++;
    }
  }

  const rmse = Math.sqrt(sqErr / testYs.length);
  const mae  = absErr / testYs.length;
  const mape = (pctErr / testYs.length) * 100;
  const directionalAccuracy = dirTotal > 0 ? (dirHits / dirTotal) * 100 : 0;

  // Multi-step rolling forecast
  let window = scaled.slice(-windowSize);
  const forecastScaled: number[] = [];
  for (let i = 0; i < horizon; i++) {
    const input = tf.tensor3d([window.map(v => [v])], [1, windowSize, 1]);
    const out = model.predict(input) as tf.Tensor;
    const next = (await out.data())[0]!;
    forecastScaled.push(next);
    window = [...window.slice(1), next];
    input.dispose(); out.dispose();
  }

  const lastActualDate  = prices[prices.length - 1]!.date;
  const lastActualClose = prices[prices.length - 1]!.close;
  const futureDates = nextBusinessDays(new Date(lastActualDate), horizon);
  const forecast: ForecastPoint[] = forecastScaled.map((v, i) => ({
    date: futureDates[i]!, predictedClose: denormalize(v, min, max),
  }));

  trainXTensor.dispose(); trainYTensor.dispose(); testXTensor.dispose(); model.dispose();

  return {
    modelName: "LSTM (32 units, 1 layer)",
    forecast, backtest,
    metrics: { rmse, mae, mape, directionalAccuracy, trainSamples: trainCount, testSamples: testCount, epochs, windowSize },
    lastActualClose, lastActualDate,
  };
}
import { Router, type IRouter, type Request, type Response } from "express";
import { z } from "zod";
import {
  GetStockHistoryParams, GetStockQuoteParams, PredictStockPriceParams,
} from "@workspace/api-zod";
import {
  defaultDateRange, fetchHistory, fetchQuote, logYahooError, summarize,
} from "../lib/stockData";
import { predictPrices } from "../lib/lstmModel";

// ---- Custom query schemas (orval's zod.date() rejects ISO date strings) ----
const optionalDate = z
  .string()
  .optional()
  .transform((v) => (v ? new Date(v) : undefined))
  .refine((d) => d === undefined || !Number.isNaN(d.getTime()), { message: "Invalid date" });

const HistoryQuerySchema = z.object({ startDate: optionalDate, endDate: optionalDate });
const PredictQuerySchema = z.object({
  startDate: optionalDate,
  endDate: optionalDate,
  horizon: z.union([z.string(), z.number()]).optional()
    .transform((v) => (v == null ? 14 : Number(v)))
    .refine((n) => Number.isFinite(n) && n >= 1 && n <= 60, { message: "horizon must be 1-60" }),
});

const router: IRouter = Router();

const POPULAR_TICKERS = [
  { ticker: "AAPL",  name: "Apple Inc." },
  { ticker: "MSFT",  name: "Microsoft Corp." },
  { ticker: "NVDA",  name: "NVIDIA Corp." },
  { ticker: "GOOGL", name: "Alphabet Inc." },
  { ticker: "AMZN",  name: "Amazon.com Inc." },
  { ticker: "TSLA",  name: "Tesla Inc." },
  { ticker: "META",  name: "Meta Platforms" },
  { ticker: "JPM",   name: "JPMorgan Chase" },
];

function sendNotFound(res: Response, ticker: string, err: unknown) {
  logYahooError(err, ticker);
  res.status(404).json({
    error: "TICKER_NOT_FOUND",
    message: `No data available for ticker "${ticker}". Please check the symbol and try again.`,
  });
}

router.get("/stocks/popular", async (req, res) => {
  try {
    const results = await Promise.allSettled(
      POPULAR_TICKERS.map(async (e) => {
        const q = await fetchQuote(e.ticker);
        return { ticker: e.ticker, name: e.name, price: q.regularMarketPrice, changePercent: q.regularMarketChangePercent };
      }),
    );
    res.json({
      tickers: results.filter((r): r is PromiseFulfilledResult<any> => r.status === "fulfilled").map((r) => r.value),
    });
  } catch (err) {
    req.log.error({ err }, "Failed to fetch popular tickers");
    res.status(500).json({ error: "INTERNAL_ERROR", message: "Failed to fetch popular tickers." });
  }
});

router.get("/stocks/:ticker/quote", async (req, res) => {
  const params = GetStockQuoteParams.safeParse(req.params);
  if (!params.success) { res.status(400).json({ error: "BAD_REQUEST", message: "Invalid ticker." }); return; }
  try { res.json(await fetchQuote(params.data.ticker)); }
  catch (err) { sendNotFound(res, params.data.ticker, err); }
});

router.get("/stocks/:ticker/history", async (req, res) => {
  const params = GetStockHistoryParams.safeParse(req.params);
  const query  = HistoryQuerySchema.safeParse(req.query);
  if (!params.success || !query.success) {
    res.status(400).json({ error: "BAD_REQUEST", message: "Invalid ticker or date range." }); return;
  }
  const { ticker } = params.data;
  const { startDate, endDate } = defaultDateRange(query.data.startDate, query.data.endDate);
  try {
    const prices = await fetchHistory(ticker, startDate, endDate);
    if (!prices.length) { sendNotFound(res, ticker, new Error("No price data.")); return; }
    const quote = await fetchQuote(ticker).catch(() => null);
    res.json({
      ticker: ticker.toUpperCase(),
      currency: quote?.currency ?? "USD",
      startDate: startDate.toISOString().slice(0, 10),
      endDate:   endDate.toISOString().slice(0, 10),
      prices, summary: summarize(prices),
    });
  } catch (err) { sendNotFound(res, ticker, err); }
});

router.get("/stocks/:ticker/predict", async (req, res) => {
  const params = PredictStockPriceParams.safeParse(req.params);
  const query  = PredictQuerySchema.safeParse(req.query);
  if (!params.success || !query.success) {
    res.status(400).json({ error: "BAD_REQUEST", message: "Invalid ticker, date range, or horizon." }); return;
  }
  const { ticker } = params.data;
  const horizon = query.data.horizon;
  const { startDate, endDate } = defaultDateRange(query.data.startDate, query.data.endDate);
  try {
    const prices = await fetchHistory(ticker, startDate, endDate);
    if (prices.length < 60) {
      res.status(400).json({
        error: "INSUFFICIENT_DATA",
        message: "Not enough historical data to train the model. Please widen the date range.",
      });
      return;
    }
    const quote  = await fetchQuote(ticker).catch(() => null);
    const result = await predictPrices(prices, { horizon });
    res.json({
      ticker: ticker.toUpperCase(),
      currency: quote?.currency ?? "USD",
      modelName: result.modelName,
      horizon,
      forecast: result.forecast,
      backtest: result.backtest,
      metrics:  result.metrics,
      lastActualClose: result.lastActualClose,
      lastActualDate:  result.lastActualDate,
    });
  } catch (err) {
    req.log.error({ err, ticker }, "Prediction failed");
    sendNotFound(res, ticker, err);
  }
});

export default router;
import app from "./app";
import { logger } from "./lib/logger";

const rawPort = process.env["PORT"];

if (!rawPort) {
  throw new Error("PORT environment variable is required but was not provided.");
}

const port = Number(rawPort);

if (Number.isNaN(port) || port <= 0) {
  throw new Error(`Invalid PORT value: "${rawPort}"`);
}

app.listen(port, (err) => {
  if (err) {
    logger.error({ err }, "Error listening on port");
    process.exit(1);
  }
  logger.info({ port }, "Server listening");
});
import express, { type Express } from "express";
import cors from "cors";
import pinoHttp from "pino-http";
import router from "./routes";
import { logger } from "./lib/logger";

const app: Express = express();

app.use(
  pinoHttp({
    logger,
    serializers: {
      req(req) {
        return { id: req.id, method: req.method, url: req.url?.split("?")[0] };
      },
      res(res) {
        return { statusCode: res.statusCode };
      },
    },
  }),
);
app.use(cors());
app.use(express.json());
app.use(express.urlencoded({ extended: true }));

app.use("/api", router);

export default app;
import pino from "pino";

const isProduction = process.env.NODE_ENV === "production";

export const logger = pino({
  level: process.env.LOG_LEVEL ?? "info",
  redact: [
    "req.headers.authorization",
    "req.headers.cookie",
    "res.headers['set-cookie']",
  ],
  ...(isProduction
    ? {}
    : {
        transport: {
          target: "pino-pretty",
          options: { colorize: true },
        },
      }),
});
import { createRequire } from "node:module";
import path from "node:path";
import { fileURLToPath } from "node:url";
import { build as esbuild } from "esbuild";
import esbuildPluginPino from "esbuild-plugin-pino";
import { rm } from "node:fs/promises";

globalThis.require = createRequire(import.meta.url);
const artifactDir = path.dirname(fileURLToPath(import.meta.url));

async function buildAll() {
  const distDir = path.resolve(artifactDir, "dist");
  await rm(distDir, { recursive: true, force: true });

  await esbuild({
    entryPoints: [path.resolve(artifactDir, "src/index.ts")],
    platform: "node",
    bundle: true,
    format: "esm",
    outdir: distDir,
    outExtension: { ".js": ".mjs" },
    logLevel: "info",
    // IMPORTANT: TensorFlow ships native shims; keep it external so the
    // pure-JS `@tensorflow/tfjs` package is loaded at runtime instead of
    // being pulled into the bundle.
    external: [
      "*.node", "sharp", "better-sqlite3", "sqlite3", "canvas",
      "bcrypt", "argon2", "fsevents", "pg-native",
      "@tensorflow/*",
      // ... other native/optional packages
    ],
    sourcemap: "linked",
    plugins: [esbuildPluginPino({ transports: ["pino-pretty"] })],
    banner: {
      js: `
import { createRequire as __bannerCrReq } from 'node:module';
import __bannerPath from 'node:path';
import __bannerUrl from 'node:url';
globalThis.require = __bannerCrReq(import.meta.url);
globalThis.__filename = __bannerUrl.fileURLToPath(import.meta.url);
globalThis.__dirname = __bannerPath.dirname(globalThis.__filename);
      `,
    },
  });
}

buildAll().catch((err) => { console.error(err); process.exit(1); });
export * from "./generated/api";

FRONTEND
{
  "name": "@workspace/stock-predictor",
  "type": "module",
  "scripts": {
    "dev":   "vite --config vite.config.ts --host 0.0.0.0",
    "build": "vite build --config vite.config.ts",
    "serve": "vite preview --config vite.config.ts --host 0.0.0.0"
  },
  "devDependencies": {
    "@tanstack/react-query": "catalog:",
    "@workspace/api-client-react": "workspace:*",
    "react": "catalog:", "react-dom": "catalog:",
    "recharts": "^2.15.2",
    "wouter": "^3.3.5",
    "lucide-react": "catalog:",
    "date-fns": "^3.6.0",
    "sonner": "^2.0.7",
    "tailwindcss": "catalog:"
    /* + shadcn UI radix-ui packages */
  }
}
import { createRoot } from "react-dom/client";
import App from "./App";
import "./index.css";

createRoot(document.getElementById("root")!).render(<App />);
import { Switch, Route, Router as WouterRouter } from "wouter";
import { QueryClient, QueryClientProvider } from "@tanstack/react-query";
import { Toaster } from "@/components/ui/toaster";
import { TooltipProvider } from "@/components/ui/tooltip";
import NotFound from "@/pages/not-found";
import Dashboard from "@/pages/Dashboard";
import { useEffect } from "react";

const queryClient = new QueryClient();

function Router() {
  return (
    <Switch>
      <Route path="/" component={Dashboard} />
      <Route component={NotFound} />
    </Switch>
  );
}

function App() {
  useEffect(() => { document.documentElement.classList.add("dark"); }, []);
  return (
    <QueryClientProvider client={queryClient}>
      <TooltipProvider>
        <WouterRouter base={import.meta.env.BASE_URL.replace(/\/$/, "")}>
          <Router />
        </WouterRouter>
        <Toaster />
      </TooltipProvider>
    </QueryClientProvider>
  );
}

export default App;
import { format } from "date-fns";

export function formatCurrency(value: number, currency = "USD", compact = false) {
  if (value == null) return "N/A";
  return new Intl.NumberFormat("en-US", {
    style: "currency", currency,
    notation: compact ? "compact" : "standard",
    maximumFractionDigits: compact ? 1 : 2,
  }).format(value);
}

export function formatNumber(value: number, compact = false) {
  if (value == null) return "N/A";
  return new Intl.NumberFormat("en-US", {
    notation: compact ? "compact" : "standard",
    maximumFractionDigits: 2,
  }).format(value);
}

export function formatPercent(value: number) {
  if (value == null) return "N/A";
  return new Intl.NumberFormat("en-US", {
    style: "percent", maximumFractionDigits: 2, signDisplay: "always",
  }).format(value / 100);
}

export function formatDate(dateStr: string) {
  if (!dateStr) return "N/A";
  return format(new Date(dateStr), "MMM d, yyyy");
}
import { useState, useMemo, useEffect } from "react";
import { Search, Activity, Cpu, AlertCircle, Info } from "lucide-react";
import {
  useGetStockQuote, useGetStockHistory, usePredictStockPrice, useGetPopularTickers,
  getGetStockHistoryQueryKey, getPredictStockPriceQueryKey, getGetStockQuoteQueryKey,
} from "@workspace/api-client-react";
import { subMonths, subYears, format } from "date-fns";
import {
  LineChart, Line, XAxis, YAxis, CartesianGrid, Tooltip as RechartsTooltip,
  Legend, ResponsiveContainer, ComposedChart, Bar,
} from "recharts";

import { Input } from "@/components/ui/input";
import { Button } from "@/components/ui/button";
import { Card, CardContent, CardHeader, CardTitle, CardDescription } from "@/components/ui/card";
import { Skeleton } from "@/components/ui/skeleton";
import { Select, SelectContent, SelectItem, SelectTrigger, SelectValue } from "@/components/ui/select";
import { Badge } from "@/components/ui/badge";
import { toast } from "sonner";
import { Tooltip, TooltipContent, TooltipProvider, TooltipTrigger } from "@/components/ui/tooltip";
import { formatCurrency, formatNumber, formatDate } from "@/lib/format";

type DateRangePreset = "6M" | "1Y" | "2Y" | "5Y";
type HorizonPreset   = 7 | 14 | 30 | 60;

function getStartDate(preset: DateRangePreset): string {
  const today = new Date();
  let start: Date;
  switch (preset) {
    case "6M": start = subMonths(today, 6); break;
    case "1Y": start = subYears(today, 1); break;
    case "2Y": start = subYears(today, 2); break;
    case "5Y": start = subYears(today, 5); break;
  }
  return format(start, "yyyy-MM-dd");
}

export default function Dashboard() {
  const [searchInput, setSearchInput] = useState("AAPL");
  const [ticker, setTicker]           = useState("AAPL");
  const [dateRange, setDateRange]     = useState<DateRangePreset>("1Y");
  const [horizon, setHorizon]         = useState<HorizonPreset>(14);

  useEffect(() => { document.title = "Stock Price Prediction Dashboard"; }, []);

  const handleSearch = (e: React.FormEvent) => {
    e.preventDefault();
    const clean = searchInput.trim().toUpperCase();
    if (!clean) { toast.error("Please enter a valid ticker symbol."); return; }
    setTicker(clean); setSearchInput(clean);
  };

  const endDate   = format(new Date(), "yyyy-MM-dd");
  const startDate = getStartDate(dateRange);

  const { data: quote,      isLoading: isLoadingQuote,      isError: isQuoteError } =
    useGetStockQuote(ticker, { query: { enabled: !!ticker, queryKey: getGetStockQuoteQueryKey(ticker), staleTime: 60_000 } });

  const { data: history,    isLoading: isLoadingHistory } =
    useGetStockHistory(ticker, { startDate, endDate }, {
      query: { enabled: !!ticker, queryKey: getGetStockHistoryQueryKey(ticker, { startDate, endDate }), staleTime: 60_000 },
    });

  const { data: prediction, isLoading: isLoadingPrediction } =
    usePredictStockPrice(ticker, { startDate, endDate, horizon }, {
      query: {
        enabled: !!ticker,
        queryKey: getPredictStockPriceQueryKey(ticker, { startDate, endDate, horizon }),
        staleTime: 5 * 60_000, gcTime: 10 * 60_000,
      },
    });

  const { data: popularTickers } = useGetPopularTickers({ query: { staleTime: 5 * 60_000 } });

  // ----- Chart data shaping -----
  const historyChartData = useMemo(() =>
    history?.prices?.map(p => ({
      date: p.date, close: p.close, volume: p.volume, displayDate: formatDate(p.date),
    })) ?? [], [history]);

  const forecastChartData = useMemo(() => {
    if (!history?.prices || !prediction?.forecast) return [];
    const lastActuals = history.prices.slice(-30).map(p => ({
      date: p.date, displayDate: formatDate(p.date),
      actual: p.close, predicted: null as number | null,
    }));
    if (lastActuals.length && prediction.forecast.length) {
      const last = lastActuals[lastActuals.length - 1];
      const fc = prediction.forecast.map(p => ({
        date: p.date, displayDate: formatDate(p.date),
        actual: null as number | null, predicted: p.predictedClose,
      }));
      last.predicted = last.actual; // connect last actual to first predicted
      return [...lastActuals, ...fc];
    }
    return [];
  }, [history, prediction]);

  const backtestChartData = useMemo(() =>
    prediction?.backtest?.map(p => ({
      date: p.date, displayDate: formatDate(p.date),
      actual: p.actualClose, predicted: p.predictedClose,
    })) ?? [], [prediction]);

  return (
    <div className="min-h-screen bg-background flex flex-col md:flex-row font-sans text-foreground">
      {/* ---------- Sidebar ---------- */}
      <aside className="w-full md:w-64 border-r border-border bg-sidebar p-4 flex flex-col gap-6 sticky top-0 h-auto md:h-screen overflow-y-auto">
        <div className="flex items-center gap-2 px-2">
          <Cpu className="text-ai w-7 h-7" />
          <span className="font-bold text-xl tracking-tight">NeuroQuant</span>
        </div>

        <form onSubmit={handleSearch} className="flex flex-col gap-3">
          <div className="relative">
            <Search className="absolute left-3 top-1/2 -translate-y-1/2 w-4 h-4 text-muted-foreground" />
            <Input value={searchInput} onChange={(e) => setSearchInput(e.target.value)}
                   placeholder="Search ticker (e.g. AAPL)"
                   className="pl-9 bg-sidebar-accent border-sidebar-border" />
          </div>
          <Button type="submit" className="w-full">Analyze</Button>
        </form>

        <div className="flex flex-col gap-2 mt-4">
          <h3 className="text-xs font-semibold text-sidebar-foreground/60 uppercase tracking-wider px-2">Watchlist</h3>
          <div className="flex flex-col gap-1">
            {popularTickers?.tickers
              ? popularTickers.tickers.map((pt) => (
                  <button key={pt.ticker}
                          onClick={() => { setTicker(pt.ticker); setSearchInput(pt.ticker); }}
                          className={`flex items-center justify-between p-2 rounded-md transition-colors ${
                            ticker === pt.ticker ? "bg-sidebar-accent/80" : "hover:bg-sidebar-accent/50 text-sidebar-foreground/80"}`}>
                    <span className="font-medium">{pt.ticker}</span>
                    <div className="flex items-center gap-2">
                      <span className="text-sm font-mono">{formatCurrency(pt.price, "USD", true)}</span>
                      <span className={`text-xs ${pt.changePercent >= 0 ? "text-emerald-500" : "text-destructive"}`}>
                        {pt.changePercent >= 0 ? "+" : ""}{pt.changePercent.toFixed(1)}%
                      </span>
                    </div>
                  </button>
                ))
              : Array.from({ length: 5 }).map((_, i) => <Skeleton key={i} className="h-10 w-full rounded-md" />)}
          </div>
        </div>

        <div className="mt-auto pt-6 text-xs text-sidebar-foreground/40 text-center px-2">
          <p>Predictions are generated by an LSTM neural network trained on historical price data. Not financial advice.</p>
        </div>
      </aside>

      {/* ---------- Main content ---------- */}
      <main className="flex-1 p-4 md:p-8 flex flex-col gap-6 overflow-y-auto w-full max-w-[1600px] mx-auto">

        {/* Header / Controls */}
        <header className="flex flex-col md:flex-row items-start md:items-center justify-between gap-4 border-b border-border pb-6">
          <div>
            <h1 className="text-3xl font-bold tracking-tight">{quote?.shortName || ticker}</h1>
            <p className="text-muted-foreground flex items-center gap-2 mt-1">
              <span>{quote?.exchange || "NASDAQ"}</span><span>•</span>
              <span className="font-mono">{ticker}</span>
            </p>
          </div>

          <div className="flex items-center gap-3 flex-wrap">
            <div className="flex items-center gap-2 bg-card border border-border rounded-lg p-1">
              {(["6M","1Y","2Y","5Y"] as DateRangePreset[]).map((p) => (
                <Button key={p} variant={dateRange === p ? "secondary" : "ghost"} size="sm"
                        onClick={() => setDateRange(p)}
                        className={`h-8 px-3 text-xs ${dateRange === p ? "font-medium" : "text-muted-foreground"}`}>
                  {p}
                </Button>
              ))}
            </div>
            <Select value={horizon.toString()} onValueChange={(v) => setHorizon(parseInt(v) as HorizonPreset)}>
              <SelectTrigger className="w-[140px] h-10"><SelectValue placeholder="Forecast Horizon" /></SelectTrigger>
              <SelectContent>
                <SelectItem value="7">7 Days</SelectItem>
                <SelectItem value="14">14 Days</SelectItem>
                <SelectItem value="30">30 Days</SelectItem>
                <SelectItem value="60">60 Days</SelectItem>
              </SelectContent>
            </Select>
          </div>
        </header>

        {/* KPI strip */}
        {isLoadingQuote ? (
          <div className="grid grid-cols-2 md:grid-cols-3 lg:grid-cols-6 gap-4">
            {Array.from({ length: 6 }).map((_, i) => <Skeleton key={i} className="h-24 rounded-xl" />)}
          </div>
        ) : isQuoteError ? (
          <div className="bg-destructive/10 text-destructive border border-destructive/20 rounded-xl p-4 flex items-center gap-3">
            <AlertCircle className="w-5 h-5" />
            <p>Failed to load quote data for {ticker}.</p>
          </div>
        ) : quote ? (
          <div className="grid grid-cols-2 md:grid-cols-3 lg:grid-cols-6 gap-4">
            {/* Current Price, Day Change, Day Range, 52W Range, Volume, Market Cap cards */}
            {/* (six <Card> tiles, each pulling from `quote`) */}
            {/* ... (omitted for brevity — see file for layout) ... */}
          </div>
        ) : null}

        {/* History + Forecast charts (2/3 + 1/3) */}
        <div className="grid grid-cols-1 lg:grid-cols-3 gap-6">
          <Card className="lg:col-span-2 flex flex-col min-h-[450px]">
            <CardHeader className="pb-2">
              <div className="flex items-center justify-between">
                <div>
                  <CardTitle>Price History & Volume</CardTitle>
                  <CardDescription>Historical performance over the selected period</CardDescription>
                </div>
                {history?.summary && (
                  <Badge variant={history.summary.totalReturnPct >= 0 ? "default" : "destructive"} className="font-mono">
                    Return: {history.summary.totalReturnPct >= 0 ? "+" : ""}{history.summary.totalReturnPct.toFixed(2)}%
                  </Badge>
                )}
              </div>
            </CardHeader>
            <CardContent className="flex-1 min-h-[350px]">
              {isLoadingHistory ? <Skeleton className="w-full h-full rounded-xl" />
              : historyChartData.length ? (
                <ResponsiveContainer width="100%" height="100%">
                  <ComposedChart data={historyChartData} margin={{ top: 10, right: 10, left: 0, bottom: 0 }}>
                    <CartesianGrid strokeDasharray="3 3" stroke="hsl(var(--border))" vertical={false} />
                    <XAxis dataKey="displayDate" stroke="hsl(var(--muted-foreground))" fontSize={12} minTickGap={50} />
                    <YAxis yAxisId="price" tickFormatter={(v) => `$${v}`} stroke="hsl(var(--muted-foreground))" fontSize={12} width={60} />
                    <YAxis yAxisId="volume" orientation="right" hide />
                    <RechartsTooltip
                      contentStyle={{ backgroundColor: "hsl(var(--card))", borderColor: "hsl(var(--border))", borderRadius: 8 }}
                      formatter={(v: number, name: string) =>
                        [name === "close" ? formatCurrency(v, history?.currency) : formatNumber(v, true),
                         name === "close" ? "Price" : "Volume"]} />
                    <Bar  yAxisId="volume" dataKey="volume" fill="hsl(var(--chart-3))" opacity={0.3} maxBarSize={20} /> **...**
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1" />
    <title>Stock Price Prediction Dashboard</title>
    <link rel="icon" type="image/svg+xml" href="/favicon.svg" />
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap" rel="stylesheet">
  </head>
  <body>
    <div id="root"></div>
    <script type="module" src="/src/main.tsx"></script>
  </body>
</html>
import { defineConfig } from "vite";
import react from "@vitejs/plugin-react";
import tailwindcss from "@tailwindcss/vite";
import path from "path";
import runtimeErrorOverlay from "@replit/vite-plugin-runtime-error-modal";

const rawPort = process.env.PORT;
if (!rawPort) throw new Error("PORT environment variable is required.");
const port = Number(rawPort);
if (Number.isNaN(port) || port <= 0) throw new Error(`Invalid PORT: "${rawPort}"`);

const basePath = process.env.BASE_PATH;
if (!basePath) throw new Error("BASE_PATH environment variable is required.");

export default defineConfig({
  base: basePath,
  plugins: [
    react(),
    tailwindcss(),
    runtimeErrorOverlay(),
    ...(process.env.NODE_ENV !== "production" && process.env.REPL_ID !== undefined
      ? [
          await import("@replit/vite-plugin-cartographer").then(m =>
            m.cartographer({ root: path.resolve(import.meta.dirname, "..") }),
          ),
          await import("@replit/vite-plugin-dev-banner").then(m => m.devBanner()),
        ]
      : []),
  ],
  resolve: {
    alias: {
      "@": path.resolve(import.meta.dirname, "src"),
      "@assets": path.resolve(import.meta.dirname, "..", "..", "attached_assets"),
    },
    dedupe: ["react", "react-dom"],
  },
  root: path.resolve(import.meta.dirname),
  build: {
    outDir: path.resolve(import.meta.dirname, "dist/public"),
    emptyOutDir: true,
  },
  server: { port, strictPort: true, host: "0.0.0.0", allowedHosts: true, fs: { strict: true } },
  preview: { port, host: "0.0.0.0", allowedHosts: true },
});
@import "tailwindcss";
@import "tw-animate-css";
@plugin "@tailwindcss/typography";

@custom-variant dark (&:is(.dark *));

@theme inline {
  --color-background: hsl(var(--background));
  --color-foreground: hsl(var(--foreground));
  --color-border: hsl(var(--border));
  --color-card: hsl(var(--card));
  --color-card-foreground: hsl(var(--card-foreground));
  --color-sidebar: hsl(var(--sidebar));
  --color-sidebar-foreground: hsl(var(--sidebar-foreground));
  --color-sidebar-accent: hsl(var(--sidebar-accent));
  --color-sidebar-border: hsl(var(--sidebar-border));
  --color-primary: hsl(var(--primary));
  --color-primary-foreground: hsl(var(--primary-foreground));
  --color-secondary: hsl(var(--secondary));
  --color-secondary-foreground: hsl(var(--secondary-foreground));
  --color-muted: hsl(var(--muted));
  --color-muted-foreground: hsl(var(--muted-foreground));
  --color-accent: hsl(var(--accent));
  --color-destructive: hsl(var(--destructive));
  --color-destructive-foreground: hsl(var(--destructive-foreground));

  /* Custom AI accent — drives the cyan glow on forecast / metrics */
  --color-ai: hsl(var(--ai));
  --color-ai-foreground: hsl(var(--ai-foreground));

  --color-chart-1: hsl(var(--chart-1));
  --color-chart-2: hsl(var(--chart-2));
  --color-chart-3: hsl(var(--chart-3));

  --font-sans: var(--app-font-sans);
  --font-mono: var(--app-font-mono);
  --radius-lg: var(--radius);
}

:root {
  /* Deep, serious financial dark mode (Bloomberg-lite) */
  --background: 224 71% 4%;
  --foreground: 213 31% 91%;
  --border:     216 34% 17%;

  --card: 224 71% 4%;
  --card-foreground: 213 31% 91%;

  --sidebar: 224 71% 4%;
  --sidebar-foreground: 213 31% 91%;
  --sidebar-border: 216 34% 17%;
  --sidebar-accent: 216 34% 17%;
  --sidebar-accent-foreground: 210 40% 98%;

  --primary: 210 40% 98%;
  --primary-foreground: 222.2 47.4% 11.2%;

  --secondary: 216 34% 17%;
  --secondary-foreground: 210 40% 98%;

  --muted: 216 34% 17%;
  --muted-foreground: 215.4 16.3% 56.9%;

  --accent: 216 34% 17%;
  --accent-foreground: 210 40% 98%;

  /* AI accent — Electric Cyan (the "neural" color) */
  --ai: 188 86% 53%;
  --ai-foreground: 222.2 47.4% 11.2%;

  --destructive: 0 62.8% 30.6%;
  --destructive-foreground: 210 40% 98%;

  /* Chart palette */
  --chart-1: 210 40% 98%;        /* white — historical prices */
  --chart-2: 188 86% 53%;        /* cyan  — AI predictions */
  --chart-3: 215.4 16.3% 56.9%;  /* gray  — volume bars */

  --app-font-sans: 'Inter', sans-serif;
  --app-font-mono: 'JetBrains Mono', monospace;
  --radius: .5rem;
}

@layer base {
  * { @apply border-border; }
  body { @apply font-sans antialiased bg-background text-foreground; }
}
{quote && (
  <div className="grid grid-cols-2 md:grid-cols-3 lg:grid-cols-6 gap-4">
    <Card className="bg-card">
      <CardContent className="p-4 flex flex-col justify-center">
        <p className="text-xs text-muted-foreground font-medium uppercase tracking-wider mb-1">Current Price</p>
        <span className="text-2xl font-bold font-mono">
          {formatCurrency(quote.regularMarketPrice, quote.currency)}
        </span>
      </CardContent>
    </Card>

    <Card className="bg-card">
      <CardContent className="p-4 flex flex-col justify-center">
        <p className="text-xs text-muted-foreground font-medium uppercase tracking-wider mb-1">Day Change</p>
        <div className="flex items-baseline gap-2">
          <span className={`text-xl font-bold font-mono ${quote.regularMarketChange >= 0 ? "text-emerald-500" : "text-destructive"}`}>
            {quote.regularMarketChange >= 0 ? "+" : ""}
            {formatCurrency(quote.regularMarketChange, quote.currency)}
          </span>
          <span className={`text-sm font-medium ${quote.regularMarketChangePercent >= 0 ? "text-emerald-500/80" : "text-destructive/80"}`}>
            ({quote.regularMarketChangePercent >= 0 ? "+" : ""}
            {quote.regularMarketChangePercent.toFixed(2)}%)
          </span>
        </div>
      </CardContent>
    </Card>

    <Card className="bg-card">
      <CardContent className="p-4 flex flex-col justify-center">
        <p className="text-xs text-muted-foreground font-medium uppercase tracking-wider mb-1">Day Range</p>
        <span className="text-lg font-semibold font-mono">
          {formatCurrency(quote.regularMarketDayLow, quote.currency, true)} - {formatCurrency(quote.regularMarketDayHigh, quote.currency, true)}
        </span>
      </CardContent>
    </Card>

    <Card className="bg-card">
      <CardContent className="p-4 flex flex-col justify-center">
        <p className="text-xs text-muted-foreground font-medium uppercase tracking-wider mb-1">52W Range</p>
        <span className="text-lg font-semibold font-mono">
          {formatCurrency(quote.fiftyTwoWeekLow, quote.currency, true)} - {formatCurrency(quote.fiftyTwoWeekHigh, quote.currency, true)}
        </span>
      </CardContent>
    </Card>

    <Card className="bg-card">
      <CardContent className="p-4 flex flex-col justify-center">
        <p className="text-xs text-muted-foreground font-medium uppercase tracking-wider mb-1">Volume</p>
        <span className="text-xl font-bold font-mono">
          {formatNumber(quote.regularMarketVolume, true)}
        </span>
      </CardContent>
    </Card>

    <Card className="bg-card">
      <CardContent className="p-4 flex flex-col justify-center">
        <p className="text-xs text-muted-foreground font-medium uppercase tracking-wider mb-1">Market Cap</p>
        <span className="text-xl font-bold font-mono">
          {quote.marketCap ? formatCurrency(quote.marketCap, quote.currency, true) : "N/A"}
        </span>
      </CardContent>
    </Card>
  </div>
)}
