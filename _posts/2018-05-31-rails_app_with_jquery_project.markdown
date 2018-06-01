---
layout: post
title:      "Rails App with jQuery project"
date:       2018-06-01 01:27:22 +0000
permalink:  rails_app_with_jquery_project
---


The project App I needed to continue to work had about 30 tables. 
And, I decided to go from the scratch for testing how it goes, and recapping the previous works. 
I did not fully complete it, but was able to do some good testing. 

At the beginning, the hard part was how to structure. 
I used the controller based asset setups, which was great for page load because I was only handling only one specific file. It made me more focused on each file, also was easy to find where the error was originated. 
I was converting the asset file extensions to `js` and `css`, so had to add each file to precompile list in `/config/initializers/assets.rb` file. It might have not been needed with coffee script and sass. It is something that I need to figure out. 


Below were some tricky part I faced with Ajax. 


1. For fetch request, my request was somehow redirected to `/users/sign_in`. However, it worked without any issue when I was hitting API with jQuery ajax method. I figured this out at the very last moment, it turned out that `credentials: "same-origin"` should be added to the parameters to pass the authentication filters. Still not clear how Rails was checking this, but error message mentinoed `before_filter` chain issues. Below is the code I completed at the end.  


```
const initWarehouseShow = () => {
  const pathname = window.location.pathname

  fetch(pathname, {
    method: "GET",
    credentials: "same-origin",
    headers: {
      "Accept": "application/json",
      "Content-type": "application/json"
    }
  })
  .then(res => res.json())
  .then((json) => {
    const warehouse = new Warehouse(json);
    const warehouseShow = warehouse.renderWarehouse();
    $("#warehouse-show").html(warehouseShow);
  })
  .catch(error => console.log(error))

}
```


2. The other part it took some time for me to figure out was how to access to multiple APIs and use them at the same time for handlebarsjs template. I did some googling, and followed as below. I declared ajax requests to the variable, and with jQuery method `when`, was able to get the mupltiple datas and handle them in one shot. 

```
    const getPaymentTerm = ajaxData("get", pathname, {}),
          getPaymentOptions = ajaxData("get", "/payment_options", {})
    $.when(getPaymentTerm, getPaymentOptions)
      .done((paymentTermData, paymentOptionsList) => {
        let data = new Object;
        data.paymentTerm = paymentTermData[0];
        data.paymentOptions = paymentOptionsList[0];
        const optionId = data.paymentTerm.payment_option.id
        data.paymentOptions.map((e) => {
          if (e.id == optionId) { e.selected = true; }
        });
        $("#content-main").html(template(data));
        $("form#payment_term").on("submit", submitPaymentTermForm);
      })
      .fail((err) => {
        console.log("err", err);
      })
```

Cheers, 

Hoon




