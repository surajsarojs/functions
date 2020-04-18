/**
 * Get Brightcove API Token Proxy
 * Requests an access token from the Brightcove OAuth API.
 *
 * @see https://support.brightcove.com/overview-oauth-api-v4
 */
import fetch from "node-fetch";
import querystring from "querystring";
const brightcoveOAuthApiUrl = "https://oauth.brightcove.com/v4/access_token";

exports.handler = async (event, context) => {
  // Only allow POST
  if (event.httpMethod !== "POST") {
    return { statusCode: 405, body: "Method Not Allowed" };
  }

  // Params are in the event body encoded as a query string
  const params = querystring.parse(event.body);
  console.log("PARAMS", params);

  // Reject if we don't have the expected parameters
  if (!(params && params.client_id && params.client_secret)) {
    return { statusCode: 401, body: "API Keys Missing" };
  }

  // Construct the auth string
  const credentials = `${params.client_id}:${params.client_secret}`;
  const authString = new Buffer(credentials).toString("base64");

  // Request an access token from Brightcove
  return fetch(brightcoveOAuthApiUrl, {
    method: "POST",
    body: "grant_type=client_credentials",
    headers: {
      Authorization: `Basic ${authString}`,
      "Content-Type": "application/x-www-form-urlencoded"
    }
  })
    .then(response => response.json())
    .then(json => ({
      statusCode: 200,
      headers: {
        "Access-Control-Allow-Origin": "*"
      },
      body: JSON.stringify(json)
    }))
    .catch(error => ({
      statusCode: 422,
      body: `Oops! Something went wrong. ${error}`
    }));
};
