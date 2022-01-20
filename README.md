# web-service-java



package gov.dukcapil.web.apps.servicesDki.dafduk.oa;

import gov.dukcapil.web.apps.servicesDki.SubVerticalServiceDki;
import gov.dukcapil.web.libs.helper.Dukcapil;
import io.vertx.core.MultiMap;
import io.vertx.core.Vertx;
import io.vertx.core.http.HttpMethod;
import io.vertx.core.json.JsonArray;
import io.vertx.core.json.JsonObject;
import io.vertx.ext.web.Router;
import io.vertx.ext.web.RoutingContext;

public class OaBiodata extends SubVerticalServiceDki {
	private static final String PKG_PROC_VIEW = "CALL SIAK_DATA.PKG_DKI_DAFDUK_BIODATA_OA.PROC_VIEW(?,?,?,?)";
	

	public OaBiodata(Vertx vertx) {
		super(vertx);
	}
	
	@Override
	protected void getRouter(Router router) {
		
		router.route(HttpMethod.GET, "/procView").handler(context -> {
			context.put(Dukcapil.REQ_STARTTIME, System.currentTimeMillis());
			procView(context);
		});
		
		
		
	}

		
	private void procView(RoutingContext context) {
		final MultiMap paramQuery = context.queryParams();
		if (!paramQuery.contains("inKey")) {
			responseBody = new JsonObject();
			responseBody.put(Dukcapil.RESP_MESSAGE, "Invalid Parameter");
			responseClientError(context, responseBody);
			return;
		}
		
		
		final JsonArray paramIn = new JsonArray();
		paramIn.add(Long.parseLong(paramQuery.get("inKey")));
		
		final JsonArray paramOut = new JsonArray();
		paramOut.addNull();
		paramOut.add(java.sql.Types.SMALLINT);
		paramOut.add(java.sql.Types.VARCHAR);
		paramOut.add(java.sql.Types.VARCHAR);		
		
		dbService.callQuery(PKG_PROC_VIEW, paramIn, paramOut, res -> {
			if (res.succeeded()) {
				JsonArray resultDb = res.result();
				int status =resultDb.getInteger(1);
				if (status != Dukcapil.RESP200) {
					responseBody = new JsonObject();
					responseBody.put(Dukcapil.RESP_MESSAGE, resultDb.getString(2));
					responseClientError(context, status, responseBody);
				}else {
					responseBody=new JsonObject(resultDb.getString(3));
					responseClientSuccess(context, responseBody);
				}
			}else{
				responseBody = new JsonObject();
				responseBody.put(Dukcapil.RESP_MESSAGE, res.cause().getMessage());
				responseClientError(context, responseBody);
			}

		});
	}
	
}



