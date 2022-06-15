import 'dart:convert';
import 'dart:math';
import 'package:avon/screens/buy_plan_success.dart';
import 'package:avon/screens/enrollee/dashboard/orders.dart';
import 'package:avon/screens/payment/paystack_screen.dart';
import 'package:avon/screens/welcome.dart';
import 'package:avon/utils/services/general.dart';
import 'package:avon/utils/services/notifications.dart';
import 'package:avon/models/buy_plan.dart';
import 'package:avon/screens/beneficiary/contact_details.dart';
import 'package:avon/screens/webviews/flutterwave-payment.dart';
import 'package:avon/state/main-provider.dart';
import 'package:avon/utils/services/http-service.dart';
import 'package:avon/utils/services/validation-service.dart';
import 'package:avon/widgets/alert_dialog.dart';
import 'package:avon/widgets/forms/text_button.dart';
import 'package:avon/widgets/scaffolds.dart';
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';
import 'package:http/http.dart' as http;
import 'package:seerbit_flutter/new/customization.dart';
import 'package:seerbit_flutter/new/methods.dart';
import 'package:seerbit_flutter/new/payload.dart';

enum PaymentMethod { FLUTTERWAVE, PAYSTACK }

class PaymentOption extends StatefulWidget {
  Map data;
  PaymentOption({Key? key, required this.data}) : super(key: key);

  @override
  _PaymentOptionState createState() => _PaymentOptionState();
}

class _PaymentOptionState extends State<PaymentOption> {
  PaymentMethod? _method;
  bool isLoading = false;

  @override
  Widget build(BuildContext context) {
    return AVScaffold(
      decoration: BoxDecoration(color: Colors.white),
      showAppBar: true,
      title: "Payment Method",
      child: WillPopScope(
        child: Padding(
          padding: const EdgeInsets.symmetric(horizontal: 20.0),
          child: Column(
            children: [
              Text(
                "Pay for your plan through any of the platforms displayed below.",
                style: TextStyle(
                  fontSize: 16,
                  fontWeight: FontWeight.w300,
                ),
              ),
              const SizedBox(height: 10),
              //paymentOpt(
              //  image: "assets/images/image 878.png",
              ////  title: "Pay with Flutterwave",
              //  mthd:  PaymentMethod.FLUTTERWAVE,
              // ),
              paymentOpt(
                  image: "assets/images/paystack.png",
                  title: "Pay with Seerbit",
                  mthd: PaymentMethod.PAYSTACK,
                  scale: 14)
            ],
          ),
        ),
        onWillPop: () async {
          return !isLoading;
        },
      ),
      bottomNavigationBar: Container(
          width: MediaQuery.of(context).size.width,
          height: 55,
          padding: EdgeInsets.symmetric(horizontal: 20),
          margin: EdgeInsets.only(bottom: 20),
          child: AVTextButton(
              radius: 5,
              child: Text('Continue',
                  style: TextStyle(color: Colors.white, fontSize: 16)),
              verticalPadding: 17,
              disabled: isLoading || _method == null,
              showLoader: isLoading,
              callBack: () {
                _continue(context);
              })),
    );
  }

  Widget paymentOpt(
          {required String image,
          required String title,
          required PaymentMethod mthd,
          double scale = 1}) =>
      InkWell(
        child: Container(
          padding: const EdgeInsets.symmetric(horizontal: 20, vertical: 20),
          margin: const EdgeInsets.symmetric(vertical: 5),
          child: Row(
            children: [
              Image.asset(
                image,
                scale: scale,
              ),
              SizedBox(
                width: 15,
              ),
              Text(title,
                  textAlign: TextAlign.start,
                  style: TextStyle(
                      fontSize: 17,
                      fontWeight: FontWeight.w600,
                      color: Colors.black)),
            ],
          ),
          decoration: BoxDecoration(
              color: mthd == _method
                  ? Color(0xff85369B).withOpacity(0.2)
                  : Colors.grey.shade50,
              borderRadius: BorderRadius.circular(10)),
        ),
        onTap: () {
          setState(() {
            _method = mthd;
          });
        },
      );

  _continue(BuildContext context) {
    MainProvider state = Provider.of<MainProvider>(context, listen: false);

    SeerbitMethod.startPayment(context,
        payload: PayloadModel(
            currency: 'NGN',
            email: !state.isLoggedIn ? widget.data['email'] : state.user.email,
            description: state.currentPlanData!['plan'].planTypeName,
            fullName:
                "${!state.isLoggedIn ? widget.data['firstName'] : state.user.firstName} ${!state.isLoggedIn ? widget.data['lastName'] : state.user.lastName}",
            country: "NG",
            transRef: DateTime.now().toString(),
            amount: widget.data['amount'].toString(),
            callbackUrl: "your callback url",
            publicKey: "SBTESTPUBK_N0y7tPQ3UzN8mJq47KchHINyQBjTwJBi",
            closeOnSuccess: false,
            closePrompt: false,
            setAmountByCustomer: false,
            pocketRef:
                "${widget.data['orderReference']}-${DateTime.now().second}",
            vendorId:
                !state.isLoggedIn ? widget.data['email'] : state.user.email,
            customization: CustomizationModel(
              borderColor: "#000000",
              backgroundColor: "#631293",
              buttonColor: "#631293",
              paymentMethod: [
                PayChannel.card,
                PayChannel.account,
                PayChannel.transfer
              ],
              confetti: false,
              logo: "logo_url || base64",
            )), onSuccess: (response) {
      print(response);
      if (response['message'] == "Successful") {
        handleResponse(response);
      }
      Navigator.pop(context);
    }, onCancel: (_) {});
  }

  _paystackPayment() {
    MainProvider _state = Provider.of<MainProvider>(context, listen: false);
    double _amount = widget.data['amount'];
    Navigator.push(
        context,
        MaterialPageRoute(
            builder: (BuildContext context) => PayStackScreen(
                  amount: _amount.toInt(),
                  email: !_state.isLoggedIn
                      ? widget.data['email']
                      : _state.user.email,
                ))).then(handleResponse);
  }

  _flutterwavePayment() {
    double _amount = widget.data['amount'];
    MainProvider _state = Provider.of<MainProvider>(context, listen: false);

    Map payload = {
      'amount': _amount,
      'email': _state.user.email, // 'ahighdee2@gmail.com',,
      'name': _state.user.firstName,
      'phone': _state.user.mobilePhone,
      'reference': widget.data['orderReference'],
      'description': "Plan Purchase"
    };
    Navigator.push(
            context,
            MaterialPageRoute(
                builder: (context) => FlutterWavePayment(data: payload)))
        .then(handleResponse);
  }

  handleResponse(value) async {
    if (value == null) return;

    if (value['message'] == 'Successful') {
      setState(() {
        isLoading = true;
      });

      MainProvider state = Provider.of<MainProvider>(context, listen: false);
      List<BuyPlan> carts = state.cart;
      carts = carts.map((e) {
        e.paid = true;
        return e;
      }).toList();

      state.cart = carts;
      state.saveCart(carts);

      Map data = widget.data;

      BuyPlan plan = carts.first;
      Map payload = {
        "amount": state.getCartTotal().toString(),
        "nhisAmount": '0',
        "totalAmount": state.getCartTotal().toString(),
        "productId": plan.selectedSubPlan?.code ?? '',
        "paymentReference": value['payments']['gatewayref'],
        "paymentMethod": "seerbit",
        "orderReference": data['orderReference'],
        "updatedBy": "${!state.isLoggedIn ? data['email'] : state.user.email}"
      };
      GeneralService()
          .setStringPref('getPaymentUserFirstName', data['firstName']);
      http.Response response = await HttpServices.post(
          context, 'plans/suscribe/complete-payment', payload);
      print("payment response: ${response.body}");

      if (response.statusCode == 200) {
        Map data = jsonDecode(response.body);
        if (data['hasError'] == false) {
          try {
            if (data['data']['enrolleeId'] != null) {
              state.user.enrolleeId = data['data']['enrolleeId'];
              GeneralService().setUser(state.user.toJson());
            }
          } catch (e) {}
          String paymentUser = "";

          if (!state.isLoggedIn) {
            GeneralService()
                .getStringPref('getPaymentUserFirstName')
                .then((value) {
              paymentUser = value.toString();
            });
            showAlertDialog(
                context: context,
                type: AlertType.SUCCESS,
                navigateTo: 'welcome',
                header:
                    "Bravo ${!state.isLoggedIn ? paymentUser : Provider.of<MainProvider>(context, listen: false).user.firstName}!",
                body: "You have successfully completed your payment",
                onContinue: () {});
          } else {
            showAlertDialog(
                context: context,
                type: AlertType.SUCCESS,
                header:
                    "Bravo ${!state.isLoggedIn ? widget.data['firstName'] : Provider.of<MainProvider>(context, listen: false).user.firstName}!",
                body: "You have successfully completed your payment",
                onContinue: () {});
          }
          if (!state.isLoggedIn) {
            Navigator.pushReplacement(context,
                MaterialPageRoute(builder: (context) => WelcomeScreen()));
          }
          if (!state.currentPlanData!['isSponsor']) {
            if (state.isLoggedIn) {
              Navigator.pushReplacement(
                  context,
                  MaterialPageRoute(
                      builder: (context) => ContactDetailsScreen(data: data)));
            } else {
              Navigator.pushReplacement(
                  context,
                  MaterialPageRoute(
                      builder: (context) => BuyPlanSuccess(
                            firstName: data['firstName'],
                          )));
            }
          } else {
            if (state.isLastPlan) {
              NotificationService.successSheet(context, data['message']);
              Navigator.pushReplacement(context,
                  MaterialPageRoute(builder: (context) => BuyPlanSuccess()));
            } else {
              Navigator.pushReplacement(
                  context,
                  MaterialPageRoute(
                      builder: (context) => OrderScreen(),
                      settings: RouteSettings(name: "order-summary")));
            }
          }
        }
      }
      setState(() {
        isLoading = false;
      });
    } else {
      showAlertDialog(
          context: context,
          type: AlertType.ERROR,
          header:
              "Sorry! ${!Provider.of<MainProvider>(context, listen: false).isLoggedIn ? Provider.of<MainProvider>(context, listen: false).user.firstName : widget.data['firstName']}!",
          body: "Your payment wasn't successful");
    }
  }
}
