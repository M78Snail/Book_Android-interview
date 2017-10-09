ListView的优化

* 重用convertView

```
public View getView(int position, View convertView, ViewGroup parent) {
        if (convertView == null) {
            convertView = LayoutInflater.from(context).inflate(R.layout.section_list_item1, null);
        }
        TextVie wtv_name = (TextView) convertView.findViewById(R.id.contact_contactinfoitem_tv_name);
        TextVie wtv_phone = (TextView) convertView.findViewById(R.id.contact_contactinfoitem_tv_phoneNum);
        Contact Info1confo = contacts.get(position);
        if (confo != null) {//toseteveryitem'stext 
            tv_name.setText(confo.getContactName());
            tv_phone.setText(confo.getContact_Phone());
        }
        return convertView;
}

```

* 使用ViewHolder
* 图片三级缓存
* 监听滑动事件
* 少用透明View
* 开启硬件加速



