Raman_messagingJSONCode
=======================

My android app
hello...my android code is

MainActivity.java

package com.example.messging_json;

import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStream;
import java.io.InputStreamReader;
import java.util.ArrayList;

import org.apache.http.HttpEntity;
import org.apache.http.HttpResponse;
import org.apache.http.client.ClientProtocolException;
import org.apache.http.client.HttpClient;
import org.apache.http.client.methods.HttpGet;
import org.apache.http.impl.client.DefaultHttpClient;
import org.json.JSONArray;
import org.json.JSONException;
import org.json.JSONObject;

import android.app.ActionBar;
import android.app.Activity;
import android.app.ProgressDialog;
import android.content.Context;
import android.graphics.drawable.ColorDrawable;
import android.os.AsyncTask;
import android.os.Bundle;
import android.text.Html;
import android.util.Log;
import android.view.LayoutInflater;
import android.view.View;
import android.view.View.OnClickListener;
import android.view.ViewGroup;
import android.widget.ArrayAdapter;
import android.widget.Button;
import android.widget.ImageView;
import android.widget.ListView;
import android.widget.TextView;
import android.widget.Toast;

public class MainActivity extends Activity {

	int listsize;
	int jarrlen;
	int newpos;
	String finalmsg;
	String builderresponse = "";
	String json = null;
	MyAdapter madapter;
	ListView lv;
	JSONArray jarrfinal;
	View newview;
	boolean iv_visibility;
	boolean iv2_visibility;
	int positionstore;
	ProgressDialog progress;
	ArrayList<Model_class> statelist;
	ArrayList<Model_class> arrnewlist;
	@Override
	protected void onCreate(Bundle savedInstanceState) {
		// TODO Auto-generated method stub
		super.onCreate(savedInstanceState);

		setContentView(R.layout.messaging);
		ActionBar actionBar = getActionBar();
		actionBar.setBackgroundDrawable(new ColorDrawable(0xFFFF4200));
		actionBar.setTitle(Html.fromHtml("<font color='#ffffff'>Messaging_JSON </font>"));
		 statelist = new ArrayList<Model_class>();
		new PostTask().execute();
		
	        //mList.setAdapter(mAdapter);
		Button button = (Button) findViewById(R.id.button1);
		button.setOnClickListener(new OnClickListener() {

			@Override
			public void onClick(View v) {
				// TODO Auto-generated method stub
				Log.d("finalmsg is", ""+finalmsg);
				try {
					 jarrfinal=new JSONArray(finalmsg);
					Log.d("finalmsg in button is", ""+jarrfinal);
					Log.d("finalmsg size in button is", ""+jarrfinal.length());
				} catch (JSONException e) {
					// TODO Auto-generated catch block
					e.printStackTrace();
				}

				if(arrnewlist.size()==jarrfinal.length()){
				android.telephony.SmsManager smsManager = android.telephony.SmsManager
						.getDefault();
				ArrayList<String> parts = smsManager.divideMessage(finalmsg);
				smsManager.sendMultipartTextMessage("09212603030", null, parts,
						null, null);
				Toast.makeText(getApplicationContext(), "SMS Delivered", Toast.LENGTH_SHORT).show();
				}
				else{
					Toast.makeText(getApplicationContext(), "Select your response for all options", Toast.LENGTH_SHORT).show();
				}
			}
		});

	}



	private class PostTask extends
			AsyncTask<String, Void, ArrayList<Model_class>> {

		@Override
		protected void onPreExecute() {
			// TODO Auto-generated method stub
			super.onPreExecute();
			progress = ProgressDialog.show(MainActivity.this, "Wait..",
					"Data is Loading...", true);
		}

		protected ArrayList<Model_class> doInBackground(String... parameters) {

			ArrayList<Model_class> finallist = new ArrayList<Model_class>();
			InputStream is = null;
			int index = 0;
			String APIurl = "http://54.169.10.2:8000/hello/";
			HttpClient httpclient = new DefaultHttpClient();
			HttpGet httpget = new HttpGet(APIurl);
			// HttpPost httppost = new HttpPost(APIurl);

			try {
				// Execute HTTP Post Request
				HttpResponse response = httpclient.execute(httpget);
				HttpEntity httpEntity = response.getEntity();
				is = httpEntity.getContent();
				BufferedReader reader = new BufferedReader(
						new InputStreamReader(is));

				String line = null;
				while ((line = reader.readLine()) != null) {

					builderresponse += line;
				}
				is.close();

				Log.d("StringBuilder...", "json is" + builderresponse);
				try {
					JSONArray jsonarr=new JSONArray(builderresponse);
					int arrlength=jsonarr.length();
					for(int g=0;g<arrlength;g++){
						int quantity=jsonarr.getJSONObject(g).getInt("quantity");
						String name=jsonarr.getJSONObject(g).getString("name");
						int id=jsonarr.getJSONObject(g).getInt("id");
						Boolean state=jsonarr.getJSONObject(g).getBoolean("state");
						Model_class mclass = new Model_class();
						mclass.setId(id);
						mclass.setName(name);
						mclass.setQuantity(quantity);
						mclass.setImagenew(android.R.drawable.btn_dialog);
						mclass.setImagenew1(android.R.drawable.checkbox_on_background);
						Log.d("quantity is", "" + mclass.getQuantity());
						Log.d("id is", "" + mclass.getId());
						Log.d("name is", "" + mclass.getName());
						finallist.add(mclass);
					}

					Log.d("arrlen", "" + arrlength);


				} catch (Exception e) {
					Log.e("Buffer Error",
							"Error converting result " + e.toString());
				}

			} catch (ClientProtocolException e) {
				// TODO Auto-generated catch block
			} catch (IOException e) {
				// TODO Auto-generated catch block
			}
			listsize = finallist.size();
			Log.d("listsize is", ""+listsize);
			return finallist;
		}

		@Override
		protected void onPostExecute(ArrayList<Model_class> result) {
			// TODO Auto-generated method stub
			Log.d("Json in postexecute is", "json postexecute is" + result);
			int visi_list_size = result.size();
			ArrayList<Boolean> visibilitylist = new ArrayList<Boolean>();
			ArrayList<Boolean> visibilitylist2 = new ArrayList<Boolean>();
			for (int j = 0; j < visi_list_size; j++) {
				visibilitylist.add(false);
				visibilitylist2.add(false);
			}

			lv = (ListView) findViewById(android.R.id.list);
			madapter = new MyAdapter(MainActivity.this,
					android.R.layout.simple_list_item_1, visibilitylist,
					visibilitylist2, result);
			lv.setAdapter(madapter);
			progress.dismiss();
			madapter.notifyDataSetChanged();

		}
	}

	private class MyAdapter extends ArrayAdapter<Model_class> {
		int count = 0;
		JSONobj_array jsonutil;
		
		ArrayList<Boolean> visibilelist;
		ArrayList<Boolean> visibilelist2;
		int i = 0;

		public MyAdapter(Context context, int resource,
				ArrayList<Boolean> visibilitylist,
				ArrayList<Boolean> visibilitylist2,
				ArrayList<Model_class> result) {
			super(context, resource);
			jsonutil = new JSONobj_array();
			arrnewlist = result;
			visibilelist = visibilitylist;
			visibilelist2 = visibilitylist2;
			// TODO Auto-generated constructor stub
		}

		@Override
		public Model_class getItem(int position) {
			// TODO Auto-generated method stub
			return arrnewlist.get(position);
		}

		@Override
		public int getCount() {
			// TODO Auto-generated method stub
			return arrnewlist.size();
		}

		@Override
		public long getItemId(int position) {
			// TODO Auto-generated method stub
			return arrnewlist.get(position).getId();
		}

		private class Viewholder {
			ImageView iv, iv2;
			TextView tv1, tv2, tv3;

		}

		@Override
		public View getView(final int position, final View convertView,
				ViewGroup parent) {
			// TODO Auto-generated method stub
			View row = convertView;
			// newview=row;
			final Viewholder holder;

			if (row == null) {

				LayoutInflater inflater = (LayoutInflater) getContext()
						.getSystemService(Context.LAYOUT_INFLATER_SERVICE);
				row = inflater.inflate(R.layout.design_list, parent, false);

				holder = new Viewholder();
				holder.tv1 = (TextView) row.findViewById(R.id.textView1);
				holder.tv2 = (TextView) row.findViewById(R.id.textView2);
				holder.tv3 = (TextView) row.findViewById(R.id.textView4);
				holder.iv = (ImageView) row.findViewById(R.id.imageView3);
				holder.iv2 = (ImageView) row.findViewById(R.id.imageView4);

				Log.d("row null block", "convertview null");
				// Log.d("gettting ids of textview tv1", ""+holder.tv1.getId());
				row.setTag(holder);
			}

			else {

				holder = (Viewholder) row.getTag();
			}
			Log.d("getview:", "" + position + " " + row);
			if ((visibilelist.get(position) == false)
					&& (visibilelist2.get(position) == false)) {
				holder.iv.setVisibility(View.VISIBLE);
				holder.iv2.setVisibility(View.VISIBLE);
			} else if ((visibilelist.get(position) == true)
					&& (visibilelist2.get(position) == false)) {
				holder.iv.setVisibility(View.INVISIBLE);
				holder.iv2.setVisibility(View.VISIBLE);
			} else if ((visibilelist2.get(position) == true)
					&& (visibilelist.get(position) == false)) {
				holder.iv2.setVisibility(View.INVISIBLE);
				holder.iv.setVisibility(View.VISIBLE);
			}

			final Model_class st = arrnewlist.get(position);
			Log.d("MODELCLASS...", "" + st.id + " " + st.name + " "
					+ st.quantity);

			holder.iv2.setImageResource(st.getImagenew());
			holder.iv.setImageResource(st.getImagenew1());
			holder.iv.setTag(new Integer(position));
			Log.d("imageview for row is clicked", "" + holder.iv.getTag());
			holder.iv2.setTag(new Integer(position));
			holder.tv1.setText("" + st.id);
			holder.tv2.setText("" + st.name);
			holder.tv3.setText("" + st.quantity);

			holder.iv2.setOnClickListener(new OnClickListener() {

				@Override
				public void onClick(View v) {
					// TODO Auto-generated method stub
					// st.setImage2(true);
					String str = Integer.toString(position);
					Log.d("str is", str);
					if ((holder.iv2.getTag().toString() == str)
							&& (visibilelist.get(position) == false)) {
						st.setWrongsate(true);
						visibilelist2.set(position, true);

						holder.iv2.setVisibility(View.INVISIBLE);
						JSONObject obj = new JSONObject();
						try {
							obj.put("id", st.id);
							obj.put("name", st.name);
							obj.put("state", "False");
							obj.put("quantity", st.quantity);
							Log.d("jsonobj is", obj.toString());
							jsonutil.putJSONobj(obj);
							finalmsg = jsonutil.getJSONArray().toString();
							Log.d("json in cross image", finalmsg);

						} catch (JSONException e) {
							// TODO Auto-generated catch block
							e.printStackTrace();
						}
					} else {
						Toast.makeText(getApplicationContext(),
								"Choose only one option ", Toast.LENGTH_SHORT)
								.show();
					}
				}
			});
			holder.iv.setOnClickListener(new OnClickListener() {

				@Override
				public void onClick(View v) {
					String str = Integer.toString(position);
					Log.d("str is", str);
					if ((holder.iv.getTag().toString() == str)
							&& (visibilelist2.get(position) == false)) {
						st.setTickstate(true);
						visibilelist.set(position, true);
						holder.iv.setVisibility(View.INVISIBLE);

						count++;
						try {
							JSONObject obj = new JSONObject();
							obj.put("id", st.id);
							obj.put("name", st.name);
							obj.put("state", "True");
							obj.put("quantity", st.quantity);
							Log.d("jsonobj is", obj.toString());
							jsonutil.putJSONobj(obj);
						   
							finalmsg = jsonutil.getJSONArray().toString();
							Log.d("json", finalmsg);
						} catch (JSONException e) {
							// TODO Auto-generated catch block
							e.printStackTrace();
						}
					} else {
						Toast.makeText(getApplicationContext(),
								"Choose only one option ", Toast.LENGTH_SHORT)
								.show();
					}
				}
			});
			
			return row;

		}

	}
}
