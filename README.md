import 'dart:io';
import 'package:avon/models/plan.dart';
import 'package:image_cropper/image_cropper.dart';
import 'package:avon/screens/payment/payment_option.dart';
import 'package:avon/state/main-provider.dart';
import 'package:avon/utils/services/http-service.dart';
import 'package:avon/utils/services/notifications.dart';
import 'package:avon/utils/services/validation-service.dart';
import 'package:avon/widgets/design/design_widget/header_progress.dart';
import 'package:avon/widgets/forms/dropdown_input.dart';
import 'package:avon/widgets/forms/text_button.dart';
import 'package:avon/widgets/forms/text_input.dart';
import 'package:file_picker/file_picker.dart';
import 'package:dotted_border/dotted_border.dart';
import 'package:flutter/cupertino.dart';
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';

class PrincipalDetailsScreen extends StatefulWidget {
  const PrincipalDetailsScreen({Key? key}) : super(key: key);

  @override
  _PrincipalDetailsScreenState createState() => _PrincipalDetailsScreenState();
}

class _PrincipalDetailsScreenState extends State<PrincipalDetailsScreen> {
  final _formKey = new GlobalKey<FormState>();

  MainProvider? state;
  File? _image;
  bool isLoading = false;
  String? title;
  String? gender;
  String? maritalStatus;
  TextEditingController _firstNameController = new TextEditingController();
  TextEditingController _lastNameController = new TextEditingController();
  TextEditingController _dobController = new TextEditingController();
  TextEditingController _emailController = new TextEditingController();

  int get progress => (state?.currentPlanIndex ?? 0) + 1;


  @override
  void initState() {
    super.initState();

    state = Provider.of<MainProvider>(context, listen: false);
    _populate();
  }

  _populate(){
    if(
      !(state?.currentPlanData!['isSponsor'] ?? true)
      && (state?.currentPlanData!['details'] == null)
    ){
      _firstNameController.text = "${state?.user.firstName}";
      _lastNameController.text = "${state?.user.lastName}";
      _emailController.text = "highdee.ai1@gmail.com";
      title = "Mr";
      gender = "Male";
      maritalStatus = "Single";
    }else if(state?.currentEnrolleData!['details'] != null){

      Map data = state?.currentEnrolleData!['details'] ?? {};
      _firstNameController.text = data['firstName'];
      _lastNameController.text = data['surname'];
      _dobController.text = data['dateOfBirth'];
      _emailController.text = data['email'];
      title = data['title'];
      gender = data['gender'] == 'm'? "Male":"Female";
      maritalStatus = data['maritalStatus'];
      _image = File(data['image']);
    }
  }

  @override
  Widget build(BuildContext context) {

    return Scaffold(
      body: Column(
        mainAxisAlignment: MainAxisAlignment.start,
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          SizedBox(height: MediaQuery.of(context).size.height * 0.05),
          headerProgress(value: MediaQuery.of(context).size.width * (progress / (state?.allCartPlans.length ?? 1))),
          SizedBox(height: 20),
          Padding(
            padding: const EdgeInsets.symmetric(horizontal: 15),
            child: Column(
              mainAxisAlignment: MainAxisAlignment.start,
              crossAxisAlignment: CrossAxisAlignment.start,
              children: [
                Row(
                  mainAxisAlignment: MainAxisAlignment.spaceBetween,
                  children: [
                    GestureDetector(
                      child: Icon(
                        Icons.arrow_back,
                        size: 30,
                        color: Colors.black,
                      ),
                      onTap: (){
                        Navigator.pop(context);
                      },
                    ),
                    Text("Step $progress of ${state?.allCartPlans.length}",
                      style: TextStyle(
                          fontWeight: FontWeight.w600,
                          fontSize: 15,
                          color: Colors.black
                      ),)
                  ],
                ),
                SizedBox(
                  height: 10,
                ),
                Text(
                    'Enter info',
                    textAlign: TextAlign.start,
                    style: TextStyle(fontSize: 20, fontWeight: FontWeight.w700, color: Colors.black)
                ),
                SizedBox(height: 5,),
                Text(
                  "Enter the details of the plan beneficiary",
                  style: TextStyle(
                    fontSize: 16,
                    fontWeight: FontWeight.w300,

                  ),
                ),
                SizedBox(height: 5,),
                Divider(),
              ],
            ),
          ),
          Expanded(
            child: Padding(
              padding: const EdgeInsets.symmetric(horizontal: 15),
              child: Form(
                key: _formKey,
                child: ListView(
                  children: [
                    Text(
                        'Principal’s details',
                        textAlign: TextAlign.start,
                        style: TextStyle(fontSize: 20, fontWeight: FontWeight.w700, color: Colors.black)
                    ),
                    SizedBox(height: 5,),
                    Text(
                      "Upload passport photo",
                      style: TextStyle(
                        fontSize: 14,
                        fontWeight: FontWeight.w300
                      ),
                    ),
                    SizedBox(height: 10,),
                    Visibility(
                        visible: _image == null  ,
                        child:Row(
                          children: [
                            DottedBorder(
                              dashPattern: [8, 4],
                              strokeWidth: 2,
                              color: Color(0xff85369B).withOpacity(0.2),
                              child: Container(
                                height: 150,
                                width: 150,
                                color: Color(0xffE7D7EB),
                                child: IconButton(
                                  icon: Icon(Icons.add,
                                      size: 50,
                                      color: Color(0xff85369B).withOpacity(0.2)
                                  ),
                                  onPressed: (){
                                    pickFile();
                                  },
                                ),
                              ),
                            ),
                          ],
                        ),
                        replacement:
                        GestureDetector(
                          child: Stack(
                            children: [
                              Image.file(
                                File("${_image?.path}"),
                                // height: MediaQuery.of(context).size.height * .2,
                                fit: BoxFit.cover,
                                width: double.infinity,
                              ),
                              IconButton(
                                  onPressed: (){
                                    setState(() {
                                      _image= null;
                                    });
                                  },
                                  icon: CircleAvatar(
                                    backgroundColor: Colors.black,
                                    child: Icon(Icons.close, color: Colors.white),
                                  )
                              )
                            ],
                          ),
                          onTap: pickFile,
                        ),
                    ),
                    Padding(padding: EdgeInsets.only(top: 40)),
                    AVDropdown(
                        options: ["Mr", "Mrs", "Miss", "Dr", "Chief", "Sir", "Lady"],
                        value: title,
                        label: "Title",
                        onChanged: (value){
                          setState((){ title = value; });
                        },
                    ),
                    SizedBox(height: 10),
                    Column(
                      children: [
                        AVInputField(
                          label: "Surname",
                          labelText: "Ayomide ",
                          controller: _lastNameController,
                          validator: (String? v) => ValidationService.isValidString(v!, minLength: 2),
                        ),
                        SizedBox(height: 10),
                        AVInputField(
                          label: "First Name",
                          labelText: "Ayomide ",
                          controller: _firstNameController,
                          validator: (String? v) => ValidationService.isValidString(v!, minLength: 2),
                        ),
                        if(state?.currentPlanData!['isSponsor'])
                        Padding(
                          padding: const EdgeInsets.only(top: 10),
                          child: AVInputField(
                            label: "Email",
                            labelText: "email",
                            controller: _emailController,
                            validator: (String? v) => ValidationService.isValidEmail(v!),
                          ),
                        ),
                        SizedBox(height: 10),
                        AVDropdown(
                          options: ["Male", "Female"],
                          value: gender,
                          label: "Gender",
                          onChanged: (value){
                            print(value);
                            setState(() {
                              gender = value;
                            });
                          },
                        ),
                        InkWell(
                          onTap: toggleDatePicker,
                          child: AVInputField(
                            label: "Date of birth",
                            labelText: "5/27/15",
                            disabled: true,
                            controller: _dobController,
                            validator: (String? v) => ValidationService.isValidInput(v!, minLength: 5),
                          ),
                        ),
                        SizedBox(height: 10),
                        AVDropdown(
                          options: ["Married", "Single", "Divorce"],
                          value: maritalStatus,
                          label: "Marital Status",
                          onChanged: (value){
                            print(value);
                            setState(() {
                              maritalStatus = value;
                            });
                          },
                        )
                      ],
                    ),
                    Container(
                        width: MediaQuery.of(context).size.width,
                        margin: EdgeInsets.symmetric(vertical: 20),
                        child: AVTextButton(
                            radius: 5,
                            child: Text('Continue', style: TextStyle(
                                color: Colors.white,
                                fontSize: 16
                            )),
                            disabled: isLoading,
                            showLoader: isLoading,
                            verticalPadding: 17,
                            callBack: submit
                        ))
                  ],
                ),
              ),
            ),
          )
        ],
      ),
    );
  }

  pickFile()async {
    FilePickerResult? result = await FilePicker.platform.pickFiles(type: FileType.image);

    if (result != null) {
      File file = File(result.files.single.path!);
      File? croppedFile = await ImageCropper().cropImage(
          sourcePath: file.path,
          aspectRatioPresets: [
            CropAspectRatioPreset.square,
            CropAspectRatioPreset.ratio3x2,
            CropAspectRatioPreset.original,
            CropAspectRatioPreset.ratio4x3,
            CropAspectRatioPreset.ratio16x9
          ],
          androidUiSettings: AndroidUiSettings(
              toolbarTitle: 'Cropper',
              toolbarColor: Colors.deepOrange,
              toolbarWidgetColor: Colors.white,
              initAspectRatio: CropAspectRatioPreset.original,
              lockAspectRatio: false),
          iosUiSettings: IOSUiSettings(
            // minimumAspectRatio: 1.0,
          )
      );

      setState(() {
        _image = croppedFile;
      });
    } else {
      // User canceled the picker
    }
  }

  toggleDatePicker()async {
    DateTime? date = await showDatePicker(
        context: context,
        initialDate:DateTime(1995, 8),
        firstDate: DateTime(1901, 1),
        lastDate: DateTime.now().add(Duration(days: -(18 * 365)))
    );
    if(date?.day != null)
      setState(() {
        _dobController.text = "${date?.day}/${date?.month}/${date?.year}";
      });
    print(_dobController.text);
  }

  submit()async {
    if(isLoading) return;
    if(!_formKey.currentState!.validate()) return;

    if(_image == null || title == null || gender == null){
      NotificationService.errorSheet(context, "Please fill all required filled");
      return;
    }

    MainProvider state = Provider.of<MainProvider>(context, listen: false);

    if(state.currentPlanIndex == null) return;

    setState(() { isLoading = true; });
    Map? planData = state.allCartPlans[state.currentPlanIndex!];
    Plan plan = planData['plan'];

    Map payload = {
      "firstName":_firstNameController.text,
      "surname":_lastNameController.text,
      "dateOfBirth":_dobController.text,
      "title":title,
      "gender":gender == 'Male' ? 'm':'f',
      "maritalStatus":maritalStatus,
      "CREATEDBY":"${state.user.email}",
      "productId": plan.code,
      "isSponsor": planData['isSponsor'] ? "1":"0"
    };

    if(planData['isSponsor']){
      payload.addAll({
        "email":_emailController.text
      });
    }else{
      payload.addAll({
        "email":state.user.email
      });
    }

    print(payload);


    var body = await HttpServices.multipartRequest(
        url: "plans/suscribe/principal-detail${(state.currentPlanIndex == 0)? '':'/others'}",
        payload: payload,
        image: _image,
        context: context);

    print(body);
    setState(() { isLoading = false; });
    if(body['hasError']) return;


    //caching the principal details of enrollee
    state.cartData[state.currentPlanIndex!]['details'] = {
      ...payload,
      "image":_image?.path,
      "orderId":body['data']['orderId'],
      "orderReference":body['data']['orderReference'],
    };

    if(state.currentPlanIndex == 0){
      Navigator.pushReplacement(context,
          MaterialPageRoute(builder: (BuildContext context)=>
              PaymentOption(data: {
                "orderReference":body['data']['orderReference'],
                "amount":state.getCartTotal() + state.nhisAmount,
              }))
      );
    }else{
      Navigator.popUntil(context, ModalRoute.withName("order-summary"));
      NotificationService.successSheet(context, body['message']);
    }

  }
}
