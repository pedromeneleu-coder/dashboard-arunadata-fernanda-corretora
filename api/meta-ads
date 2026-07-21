const DEFAULT_TABLE = "dashboard_meta_daily_fernanda_sp";
const DEFAULT_CLIENT_ID = "fernanda_sp";
const PAGE_SIZE = 1000;
const SELECT_COLUMNS = "client_id,date,campaign_id,campaign_name,adset_id,adset_name,ad_id,ad_name,impressions,clicks,spend,leads";

function sendError(response, status, message) {
  return response.status(status).json({ error: message });
}

module.exports = async function handler(request, response) {
  if (request.method !== "GET") {
    response.setHeader("Allow", "GET");
    return sendError(response, 405, "Metodo nao permitido.");
  }

  const supabaseUrl = process.env.SUPABASE_URL || "";
  const supabaseAnonKey = process.env.SUPABASE_ANON_KEY || "";
  const tableName = process.env.SUPABASE_TABLE_NAME || DEFAULT_TABLE;
  const clientId = process.env.SUPABASE_CLIENT_ID || DEFAULT_CLIENT_ID;

  if (!/^https:\/\/[^/]+/i.test(supabaseUrl) || !supabaseAnonKey) {
    return sendError(response, 500, "Variaveis SUPABASE_URL e SUPABASE_ANON_KEY nao configuradas na Vercel.");
  }

  if (!/^[a-zA-Z_][a-zA-Z0-9_]*$/.test(tableName)) {
    return sendError(response, 500, "Nome de tabela invalido na configuracao da Vercel.");
  }

  try {
    const rows = [];
    let offset = 0;

    while (true) {
      const url = new URL(`/rest/v1/${encodeURIComponent(tableName)}`, supabaseUrl);
      url.searchParams.set("select", SELECT_COLUMNS);
      url.searchParams.set("order", "date.asc");
      url.searchParams.set("limit", String(PAGE_SIZE));
      url.searchParams.set("offset", String(offset));
      url.searchParams.set("client_id", `eq.${clientId}`);

      const supabaseResponse = await fetch(url, {
        headers: {
          apikey: supabaseAnonKey,
          Authorization: `Bearer ${supabaseAnonKey}`,
          Accept: "application/json"
        }
      });

      if (!supabaseResponse.ok) {
        return sendError(response, supabaseResponse.status, "Falha ao consultar os dados no Supabase.");
      }

      const page = await supabaseResponse.json();
      if (!Array.isArray(page)) {
        return sendError(response, 502, "O Supabase retornou uma resposta inesperada.");
      }

      rows.push(...page);
      if (page.length < PAGE_SIZE) break;
      offset += PAGE_SIZE;
    }

    response.setHeader("Cache-Control", "s-maxage=60, stale-while-revalidate=300");
    response.setHeader("X-Content-Type-Options", "nosniff");
    return response.status(200).json({ data: rows });
  } catch (error) {
    console.error("Erro ao consultar o Supabase:", error);
    return sendError(response, 500, "Erro interno ao buscar os dados do dashboard.");
  }
};
