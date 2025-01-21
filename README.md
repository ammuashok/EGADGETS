# EGADGETS
EGadgets is an on-demand e-gadget repair and support system aimed at providing efficient repair services for electronic gadgets like smartphones, laptops, tablets, and more. The system integrates physical service centers with an online platform to address common challenges .


from django.shortcuts import render,get_object_or_404,redirect
from .forms import GadgetModelform,TechModelform,Userloginform,Techloginform,Productdetailsform,productamountform,paymentform,servicestatusform,Feedbackform,chatform,Compliants,replay,Adminloginform
from .models import GadgetModel,TechModel,ProductModel,paymentModel,FeedbackModel,newchat,CompliantsModel,AdminModel
from django.db.models import Q
from operator import attrgetter
from itertools import chain
from django.http import HttpResponse
from datetime import date
from django.http import JsonResponse
from django.contrib import messages

from django.conf import settings


# Create your views here.
def indexuser(request):
    return render(request,'indexuser.html')
def adminpage(request):
    return render(request,'adminindex.html')
def login(request):
    return render(request,'loginindex.html')
def chat(request):
    return render(request,'c.html')
def rating(request):
    return render(request,'rating.html')
def about(request):
    return render(request,'aboutus.html')
def services(request):
    return render(request,'services.html')
def contact(request):
    return render(request,'contact.html')
def user_reg(request):
    if request.method=='POST':
        form=GadgetModelform(request.POST)
        if form.is_valid():
            Name= form.cleaned_data['Name']
            contact = form.cleaned_data['ContactNumber']
            email = form.cleaned_data['Email']
            password = form.cleaned_data['Password']
            if GadgetModel.objects.filter(Email=email).exists():
                messages.error(request,'This email is alredy registerd')
            
            else:
                form.save()
                subject = 'Registration Successful'
                message = f'Hello {Name},\n\nThank you for registering with us. Your registration was successful.\n\nBest regards,\nYour Team'
                email_from = settings.EMAIL_HOST_USER
                recipient_list = [email]
                
                send_mail(subject, message, email_from, recipient_list)
                
                messages.success(request, 'Registration successful. A confirmation email has been sent.')
                return redirect('index')
    else:
        form=GadgetModelform()
    return render(request,'regformtemp.html',{'form':form})  


# views.py

# views.py

# from django.shortcuts import render, redirect
# from django.core.mail import send_mail
# from django.template.loader import render_to_string
# from django.utils.html import strip_tags
# from .forms import GadgetModelform

# def user_reg(request):
#     if request.method == 'POST':
#         form = GadgetModelform(request.POST)
#         if form.is_valid():
#             user = form.save()
            
#             # Prepare email content
#             subject = 'Registration Successful'
#             html_message = render_to_string('registration_email.html', {'user': user})
#             plain_message = strip_tags(html_message)
#             from_email = 'your-email@gmail.com'  # Replace with your email
#             to_email =GadgetModelform .Email  # Assuming the form saves the email field
            
#             # Send email to the newly registered user
#             send_mail(subject, plain_message, from_email, [to_email], html_message=html_message)
            
#             # Optionally, you could also send an email to admins or other recipients if needed

#             return redirect('success')  # Redirect to a success page
#     else:
#         form = GadgetModelform()
#     return render(request, 'regformtemp.html', {'form': form})


def tech_reg(request):
    if request.method=='POST':
        form=TechModelform(request.POST,request.FILES)
        if form.is_valid():
            Name= form.cleaned_data['Name']
            Contact_Number = form.cleaned_data['Contact_Number']
            email = form.cleaned_data['Email']
            password= form.cleaned_data['Password']
            if TechModel.objects.filter(Email=email).exists():
                messages.error(request,'This email is alredy registerd')
            
            else:
                form.save()
                subject = 'Registration Successful'
                message = f'Hello {Name},\n\nThank you for registering with us. Your registration was successful.\n\nBest regards,\nYour Team'
                email_from = settings.EMAIL_HOST_USER
                recipient_list = [email]
                
                send_mail(subject, message, email_from, recipient_list)
                
                messages.success(request, 'Registration successful. A confirmation email has been sent.')
           
            return redirect('index')
    else:
        form=TechModelform()
    return render(request,'regformtemp.html',{'form':form})  
 
def adminuser(request):
    book=GadgetModel.objects.all()
    return render(request,'adminuser.html',{'book':book})
def admintech(request):
    book=TechModel.objects.all()
    return render(request,'admintech.html',{'book':book})
def User_login(request,usertype):
    if usertype=='user':
        if request.method=='POST':
            form=Userloginform(request.POST)
            if form.is_valid():
                email=form.cleaned_data['Email']
                password=form.cleaned_data['Password']
                user=GadgetModel.objects.filter(Email=email,Password=password).first()
                if user is not None:
                    request.session['user_id']=user.id
                    return redirect('indexuser')
        else:
            form=Userloginform()
        return render(request,'loginindex.html',{'loginindex':form}) 

    if usertype=='tech':
        if request.method=='POST':
            form=Techloginform(request.POST)
            if form.is_valid():
                email=form.cleaned_data['Email']
                password=form.cleaned_data['Password']
                user=TechModel.objects.filter(Email=email,Password=password,Accept_status=1).first()
                if user is not None:
                    request.session['tech_id']=user.id
                    return redirect('techindex')
                else:
                    return HttpResponse('Access denied,Your account does not have admin privilleges')
        else:
            form=Techloginform()
        return render(request,'loginindex.html',{'loginindex':form})            
def tech_index(request):
    return render(request,'techindex.html')
def index(request):
    return render(request,'index.html')
from django.http import HttpResponseForbidden
def public_update(request,user_id):
    session_user_id = request.session.get('user_id')

    if session_user_id != user_id:
        return HttpResponseForbidden("You are not authorized to access this page.")
    var=request.session.get('user_id')
    
    instance1=get_object_or_404(GadgetModel,pk=var)
    if request.method=='POST':
        form=GadgetModelform(request.POST,instance=instance1)
        if form.is_valid():
            form.save()
            return redirect('usuccess_page')
    else:
        form=GadgetModelform(instance=instance1)
    return render(request,'publicprofile.html',{'form':form})
def tech_update(request,tech_id):
    session_user_id = request.session.get('tech_id')

    if session_user_id != tech_id:
        return HttpResponseForbidden("You are not authorized to access this page.")
    var=request.session.get('tech_id')
    instance1=get_object_or_404(TechModel,pk=var)
    if request.method=='POST':
        form=TechModelform(request.POST,request.FILES,instance=instance1)
        if form.is_valid():
            form.save()
            print("File path: ", instance1.certificate.path)
            return redirect('tsuccess_page')
    else:
        form=TechModelform(instance=instance1)
    return render(request,'techprofile.html',{'form':form})
# def tech_search(request):
#     var=request.GET.get('search')
#     result=None
#     if var:
#         result=TechModel.objects.filter(Q(Specialization__icontains=var))
#     return render(request,'searchresult.html',{'result':result,'var':var})


from django.db.models import Q, Count, Case, When, IntegerField

def tech_search(request):
    var = request.GET.get('search')
    result = None

    if var:
        # Get the current user's location (city and state)
        current_user = request.session.get('user_id')
        user_info = GadgetModel.objects.get(pk=current_user)
        user_city = user_info.City
        user_district = user_info.District

        # Filter technicians based on user's city, state, and specialization
        techs = TechModel.objects.filter(
            (Q(Specialization__icontains=var) | Q(city__icontains=var)) &
            Q(Accept_status=1)  # Only include technicians who are accepted
        )

        # Annotate the technicians with the count of good feedback
        techs = techs.annotate(
            good_feedback_count=Count(
                Case(
                    When(feedbackmodel__Feedback__icontains='good', then=1),
                    When(feedbackmodel__Feedback__icontains='excellent', then=1),
                    When(feedbackmodel__Feedback__icontains='great', then=1),
                    When(feedbackmodel__Feedback__icontains='super', then=1),
                    When(feedbackmodel__Feedback__icontains='amazing', then=1),
                    When(feedbackmodel__Feedback__icontains='wonderful', then=1),
                    When(feedbackmodel__Feedback__icontains='fantastic', then=1),
                    When(feedbackmodel__Feedback__icontains='awesome', then=1),
                    When(feedbackmodel__Feedback__icontains='fantabulous', then=1),
                    When(feedbackmodel__Feedback__icontains='fabulous', then=1),
                    When(feedbackmodel__Feedback__icontains='outstanding', then=1),
                    When(feedbackmodel__Feedback__icontains='unbelievable', then=1),
                    When(feedbackmodel__Feedback__icontains='breathtaking', then=1),
                    When(feedbackmodel__Feedback__icontains='unbelievable', then=1),
                    When(feedbackmodel__Feedback__icontains='magnificent', then=1),
                    When(feedbackmodel__Feedback__icontains='kollaam', then=1),
                    When(feedbackmodel__Feedback__icontains='adipoli', then=1),
                    output_field=IntegerField(),
                )
            ),
            avg_rating=Coalesce(
                Avg('productmodel__rating'),
                Value(0),
                output_field=FloatField()
            )
        ).order_by('-good_feedback_count')

        result = techs

    return render(request, 'searchresult.html', {'result': result, 'var': var})

from django.db.models import Avg, Count, Case, When, IntegerField, FloatField, Value
from django.db.models.functions import Coalesce

# def tech_search(request):
#     var1=request.session.get('user_id')  
#     instance1=GadgetModel.objects.get(pk=var1)
#     var = request.GET.get('search')
#     result = None
#     if var:
#         # Filter technicians based on specialization
#         techs = TechModel.objects.filter(Q(Specialization__icontains=var))
#         user_location = request.GET.get('district') 
#     if user_location:
#             lat, lon = map(float, user_location.split(','))
#             techs = techs.annotate(distance=Distance('district',user_point)).filter(distance__lte=D(km=10)).order_by('distance')
        
#         # Annotate the technicians with the count of good feedback
#     techs = techs.annotate(
#             good_feedback_count=Count(
#                 Case(
#                     When(feedbackmodel__Feedback__icontains='good', then=Value(1)),
#                     When(feedbackmodel__Feedback__icontains='excellent', then=Value(1)),
#                     When(feedbackmodel__Feedback__icontains='great', then=Value(1)),
#                     When(feedbackmodel__Feedback__icontains='super', then=Value(1)),
#                     When(feedbackmodel__Feedback__icontains='amazing', then=Value(1)),
#                     When(feedbackmodel__Feedback__icontains='wonderful', then=Value(1)),
#                     When(feedbackmodel__Feedback__icontains='fantastic', then=Value(1)),
#                     When(feedbackmodel__Feedback__icontains='awesome', then=Value(1)),
#                     When(feedbackmodel__Feedback__icontains='fantabulous', then=Value(1)),
#                     When(feedbackmodel__Feedback__icontains='fabulous', then=Value(1)),
#                     When(feedbackmodel__Feedback__icontains='outstanding', then=Value(1)),
#                     When(feedbackmodel__Feedback__icontains='unbelievable', then=Value(1)),
#                     When(feedbackmodel__Feedback__icontains='breathtaking', then=Value(1)),
#                     When(feedbackmodel__Feedback__icontains='magnificent', then=Value(1)),
#                     When(feedbackmodel__Feedback__icontains='kollaam', then=Value(1)),
#                     When(feedbackmodel__Feedback__icontains='adipoli', then=Value(1)),
#                     output_field=IntegerField(),
#                 )
#             ),
#             avg_rating=Coalesce(
#                 Avg('productmodel__rating'),
#                 Value(0),
#                 output_field=FloatField()
#             )
#         ).order_by('-good_feedback_count','distance')
        
#         result = techs
    
#     return render(request, 'searchresult.html', {'result': result, 'var': var})

def product_details(request,id): 
    var1=request.session.get('user_id')  
    instance1=GadgetModel.objects.get(pk=var1)
    instance2=get_object_or_404(TechModel,pk=id)
    if request.method=='POST':
        form=Productdetailsform(request.POST)
        if form.is_valid():
            var1=form.save(commit=False)
            var1.Userid=instance1 
            var1.Techid=instance2           
            var1.save()
            return redirect('sucesss')
    else:
        form=Productdetailsform()
    return render(request,'productdeatils.html',{'form':form})

def edit_product(request,pk):
    instance=get_object_or_404(ProductModel,id=pk)
    form=Productdetailsform(request.POST or None,instance=instance)
    if form.is_valid():
        form.save()
        return redirect('prouserview')
    return render(request,'productedituser.html',{'form':form})
def remove_product(request,pk):
    instance=ProductModel.objects.get(id=pk)
    instance.delete()
    return redirect('prouserview')
def producttech_view(request):
     var=request.session.get('tech_id')
     instance1=get_object_or_404(TechModel,pk=var)
    #  form=ProductModel.objects.all()
     form=ProductModel.objects.filter(Techid=instance1)
     return render(request,'producttechview.html',{'form':form})
def publicdttech_view(request):
    # Filter out technicians with a rejected status
    book = TechModel.objects.filter(Accept_status=1)
    
    return render(request, 'publictechdview.html', {'book': book})


def productuser_view(request):
     var=request.session.get('user_id') 
     instance1=get_object_or_404(GadgetModel,pk=var)
    #  form=ProductModel.objects.all()
     form=ProductModel.objects.filter(Userid=instance1)
     return render(request,'productuserview.html',{'form':form})  

def statusaprove(request,pk):
    data=ProductModel.objects.get(pk=pk)
    data.Accept_status = 1
    data.save()
    return redirect('productview')
def statusreject(request,pk):
    data=get_object_or_404(ProductModel,pk=pk)
    data.Accept_status = 2
    data.save()
    return redirect('productview')
def paymentsuccess(request,pk):
    data=get_object_or_404(paymentModel,pk=pk)
    data.payment_status = 1
    data.save()
    return redirect('pro')
def Amount(request,pk):
    if request.method=='POST':
        product=get_object_or_404(ProductModel,pk=pk)
        form=productamountform(request.POST)
        if form.is_valid():
            v=form.cleaned_data['Amount']
            product.Amount=v
            product.save()
            return redirect('productview')
    else:
        form=productamountform()
    return render(request,'Amount.html',{'form':form}) 
def userheader(request):
    return render(request,'userheader.html')
def techheader(request):
    return render(request,'techheader.html')
def adminheader(request):
    return render(request,'adminheader.html')
def mainheader(request):
    return render(request,'mainheader.html')
def paymenttemp(request):
    return render(request,'paymenttemp.html')
def adminproduct(request):
    form=ProductModel.objects.all()
    return render(request,'adminproductview.html',{'form':form})
def adminfeedbackview(request):
   form = FeedbackModel.objects.annotate(avg_rating=Avg('productid__rating'))
   return render(request,'adminfeedbackview.html',{'form':form})



def payment(request,pk,Amount):
    var=request.session.get('user_id')
    instance=get_object_or_404(GadgetModel,pk=var)
    instance1=get_object_or_404(ProductModel,id=pk)
    amount=float(Amount)
    if request.method=='POST':
        
        form=paymentform(request.POST)
        if form.is_valid():
            var1=form.save(commit=False)
            Commision=Amount * 0.05
            var1.Userid=instance
            var1.productid=instance1
            var1.Commision=Commision
            var1.Amount=amount
            var1.save()
           
            var2=ProductModel.objects.get(id=pk)
           
            var2.paymentstatus = 1
            var2.save()
            
            return redirect('psuccess_page')
    else:
        form=paymentform()
        
    return render(request,'paymenttemp.html',{'form':form})
from django.utils import timezone
def Service_Status(request, pk):
    var=request.session.get('tech_id')
    instance=get_object_or_404(TechModel,pk=var)
    instance1= get_object_or_404(ProductModel, id=pk)
    
    
    if request.method == 'POST':
        form = servicestatusform(request.POST, instance=instance)
        if form.is_valid():
            var1 = form.save(commit=False)
            var1.productid=instance1
            
            # Update the date field based on the selected service status
            if var1.service_status == 'startrepairing':
                var1.date = timezone.now()  # The date when status is updated to Start Repairing
            elif var1.service_status == 'Processing':
                var1.processdate = timezone.now()  # The date when status is updated to Processing
            elif var1.service_status == 'readyfordelivering':
                var1.deliveringdate = timezone.now()  # The date when status is updated to Ready for Delivering
            elif var1.service_status == 'Delivered':
                var1.deliverddate = timezone.now()  # The date when status is updated to Delivered

            var1.save()
            return redirect('servicestatusview')
    else:
        form = servicestatusform(instance=instance)
    
    return render(request, 'servicestatus.html', {'form': form})           
def edit_status(request,pk):
    instance=get_object_or_404(ProductModel,id=pk)
    form=servicestatusform(request.POST or None,instance=instance)
    if form.is_valid():
        form.save()
        return redirect('servicestatusview')
    return render(request,'editstatus.html',{'form':form})
def remove_status(request,pk):
    instance=ProductModel.objects.get(id=pk)
    instance.delete()
    return redirect('servicestatusview')
# def servicestatus_view(request):
#      var=request.session.get('tech_id')
#      instance1=get_object_or_404(TechModel,pk=var)
#      form=ProductModel.objects.filter(Techid=instance1)
def servicestatus_view(request):
      var=request.session.get('tech_id') 
      instance1=get_object_or_404(TechModel,pk=var)
      form=ProductModel.objects.filter(Techid=instance1)
      return render(request,'statusview.html',{'form':form})



 
def feedbackuser_view(request):
     var=request.session.get('user_id') 
     instance1=get_object_or_404(GadgetModel,pk=var)
     form=FeedbackModel.objects.filter(Userid=instance1)
     return render(request,'feedbackuserview.html',{'form':form})

def edit_feedback(request,Feedback_id):
    instance=get_object_or_404(FeedbackModel,Feedback_id=Feedback_id)
    form=Feedbackform(request.POST or None,instance=instance)
    if form.is_valid():
        form.save()
        return redirect('prouserview')
    return render(request,'feededit.html',{'form':form})
def remove_feedback(request,Feedback_id):
    instance=FeedbackModel.objects.get(Feedback_id=Feedback_id)
    instance.delete()
    return redirect('feedbackuserview')
def feedbacktech_view(request):
    var = request.session.get('tech_id')
    instance1 = get_object_or_404(TechModel, pk=var)
    
    # Annotate the feedback with the average rating for each product
    form = FeedbackModel.objects.filter(Techid=instance1).annotate(avg_rating=Avg('productid__rating'))
    
    return render(request, 'feedbacktechview.html', {'form': form})

# def feedbackpublic_view(request):
#     form=FeedbackModel.objects.all()
#     form.objects.annotate(avg_rating=Avg('ProductModel__rating'))
   
#     return render(request,'publicpagereview.html',{'form':form})

def feedbackpublic_view(request):
    # Annotate each feedback with the average rating of the associated product
    form = FeedbackModel.objects.annotate(avg_rating=Avg('productid__rating'))
   
    # Render the template and pass the feedback list to it
    return render(request, 'publicpagereview.html', {'form':form})
def statusview(request):
     form=ProductModel.objects.all()
     return render(request,'statusview.html',{'form':form})

def chattemp(request):
    return render(request,'chattemplate.html')
def chat(request,pk):
    var=request.session.get('user_id')
    instance1=get_object_or_404(GadgetModel,pk=var)
    instance2=get_object_or_404(TechModel,pk=pk)
    if request.method =='POST':
        form=chatform(request.POST)
        if form.is_valid():
            message=form.save(commit=False)
            message.Userid=instance1
            message.Techid=instance2
            message.save()
            # message1=TechModel.objects.get(pk=id)
            # message1.save()
            return redirect('chat')
    else:
        form=chatform()
    return render(request,'chattemplate.html',{'form':form})
def chattech(request,pk):
    var=request.session.get('tech_id')
    instance1=get_object_or_404(TechModel,pk=var)
    instance2=get_object_or_404(GadgetModel,pk=pk)
    if request.method=='POST':
        form=chatform(request.POST)
        if form.is_valid():
            message=form.save(commit=False)
            message.Techid=instance1
            message.Userid=instance2
            message.save()
            return redirect('chattech')
    else:
        form=chatform()
        return render(request,'chattemplate.html',{'form':form})    
def chatnew(request,Userid=None,Techid=None):
    if  Techid:
        userid=request.session.get('user_id')
        User_sender=get_object_or_404(GadgetModel,pk=userid)
        tech_receiver=get_object_or_404(TechModel,pk=Techid)
        user_receiver=None
        tech_sender=None
    elif Userid:
        techid=request.session.get('tech_id')
        tech_sender=get_object_or_404(TechModel,pk=techid)
        user_receiver=get_object_or_404(GadgetModel,pk=Userid)
        tech_receiver=None
        User_sender=None
    else:
        return redirect('prouserview')
    if request.method =='POST':
        message=request.POST.get('messages')
        form=chatform(request.POST)

        if User_sender and tech_receiver:
            newchat.objects.create(
                messages=message,
                User_sender=User_sender,
                tech_receiver=tech_receiver
            )
            return redirect('chat',Techid=Techid)
        
        if tech_sender and user_receiver:
            newchat.objects.create(
                messages=message,
                tech_sender=tech_sender,
                user_receiver=user_receiver
            )
            return redirect('chattech',Userid=Userid)
    else:
        form=chatform()
    if User_sender and tech_receiver:
        sent_msg=newchat.objects.filter(User_sender=User_sender,tech_receiver=tech_receiver)
        recieved_msg=newchat.objects.filter(tech_sender=tech_receiver,user_receiver=User_sender)
    elif tech_sender and user_receiver:
        sent_msg=newchat.objects.filter(tech_sender=tech_sender,user_receiver=user_receiver)
        recieved_msg=newchat.objects.filter(User_sender=user_receiver,tech_receiver=tech_sender)
    smessages=sent_msg
    rmessages=recieved_msg
    all_messages =sorted(
        chain(rmessages,smessages),
        key=attrgetter('current_date')
    )  
    if User_sender:
        context={
            'message':all_messages,
            'User_id':User_sender,
            'user_receiver':user_receiver,
            'tech_receiver':tech_receiver,
            'form':form
        
        } 
    if tech_sender:
        context={
            'message':all_messages,
            'User_id':tech_sender,
            'user_reciver':user_receiver,
            'tech_receiver':tech_receiver,
            'form':form
        
        }  
    return render(request,'chattemplate.html',context)           

 
          
def compliantuser_view(request):
     var=request.session.get('user_id') 
     instance1=get_object_or_404(GadgetModel,pk=var)
     form=CompliantsModel.objects.filter(Userid=instance1)
     return render(request,'ch.html',{'form':form})
def edit_compliants(request,Compliant_id):
    instance=get_object_or_404(CompliantsModel,Compliant_id=Compliant_id)
    form=Compliants(request.POST or None,instance=instance)
    if form.is_valid():
        form.save()
        return redirect('compliantuserview')
    return render(request,'compliantedit.html',{'form':form})
def remove_compliants(request,Compliant_id):
    instance1=CompliantsModel.objects.get(Compliant_id=Compliant_id)
    instance1.delete()
    return redirect('compliantuserview')
def admincompliantview(request):
    form=CompliantsModel.objects.all()
    return render(request,'admincompliantview.html',{'form':form})               
def usercompliantview(request):
     var=request.session.get('user_id') 
     instance1=get_object_or_404(GadgetModel,pk=var)
     form=CompliantsModel.objects.filter(Userid=instance1)
     return render(request,'ch.html',{'form':form})
def complaintreplay(request,Compliant_id):
   if request.method=='POST':
        Compliant=get_object_or_404( CompliantsModel,pk=Compliant_id)
        form= replay(request.POST)
        if form.is_valid():
            v=form.cleaned_data['Replay']
            Compliant.Replay=v
            Compliant.save()
            return redirect('adminpage')
   else:
        form=replay()
   return render(request,'adminreply.html',{'form':form})
def edit_replay(request,Compliant_id):
    instance=get_object_or_404(CompliantsModel,Compliant_id=Compliant_id)
    form= replay(request.POST or None,instance=instance)
    if form.is_valid():
        form.save()
        return redirect('compliantuserview')
    return render(request,'replayedit.html',{'form':form})


def commisionview(request):
    form=paymentModel.objects.all()
    return render(request,'adminpaymentstatus.html',{'form':form}) 

def techaprove(request,pk):
    data=TechModel.objects.get(pk=pk)
    data.Accept_status = 1
    data.save()
    return redirect('admintech')
def techreject(request,pk):
    data=TechModel.objects.get(pk=pk)
    data.Accept_status = 2
    data.save()
    return redirect('admintech')
 
from django.shortcuts import render
from .models import ProductModel
from datetime import date, timedelta

from django.shortcuts import render, get_object_or_404
from .models import ProductModel
from datetime import date, timedelta
from datetime import date, timedelta
from django.shortcuts import render, get_object_or_404
from .models import ProductModel
from django.shortcuts import render, get_object_or_404
from .models import ProductModel
from datetime import date

def trackorder(request, pk):
    product = get_object_or_404(ProductModel, id=pk)
    new_date = date.today()
    orders = ProductModel.objects.filter(Current_Date=product.Current_Date)  # Filter based on the order placed date
    tracking_data = []

    # Define the stages and their progress thresholds
    stages = {
        'Start Repairing': 33,
        'Processing': 50,
        'readyfordelivering': 66,
        'Delivered': 100
    }

    status_to_progress = {
        'startrepairing': 10,
        'Processing': 50,
        'readyfordelivering': 75,
        'Delivered': 100
    }

    for order in orders:
        # Calculate the progress based on service status
        progress = status_to_progress.get(order.service_status, 0)
        
        # Determine the progress steps
        progress_steps = sum(
            1 for stage, threshold in stages.items()
            if progress >= threshold
        )

        tracking_data.append({
            'product_name': order.Product_Name,
            'progress': progress,
            'progress_steps': progress_steps,
            'tracking_steps': [
                {"status": "Start Repairing", "date": order.date.strftime('%Y-%m-%d') if order.date else 'N/A'},
                {"status": "Processing", "date": order.processdate.strftime('%Y-%m-%d') if order.processdate else 'N/A'},
                {"status": "Ready for Delivering", "date": order.deliveringdate.strftime('%Y-%m-%d') if order.deliveringdate else 'N/A'},
                {"status": "Delivered", "date": order.deliverddate.strftime('%Y-%m-%d') if order.deliverddate else 'N/A'}
            ]
        })

    return render(request, 'trackorder.html', {'tracking_data': tracking_data, 'current_date': new_date})




# def trackorder(request, pk):
#     product = get_object_or_404(ProductModel, id=pk)
#     new_date = date.today()
#     orders = ProductModel.objects.filter(Current_Date=product.Current_Date)  # Filter based on the order placed date
#     tracking_data = []

#     for order in orders:
#         order_placed = order.Current_Date
#         StartRepairing= order_placed + timedelta(days=2)
#         out_for_delivery =  StartRepairing+ timedelta(days=7)
#         delivered = out_for_delivery + timedelta(days=8)

#         tracking_data.append({
#             'product_name': order.Product_Name,
#             'tracking_steps': [
#                 {"status": "Order Placed", "date": order_placed.strftime('%Y-%m-%d')},
#                 {"status": " Start Repairing", "date":  StartRepairing.strftime('%Y-%m-%d')},
#                 {"status": "Out for Delivery", "date": out_for_delivery.strftime('%Y-%m-%d')},
#                 {"status": "Delivered", "date": delivered.strftime('%Y-%m-%d')}
#             ]
#         })

#     return render(request, 'trackorder.html', {'tracking_data': tracking_data, 'current_date': new_date})




# from django.shortcuts import render, get_object_or_404, redirect
# from django.db.models import Avg
# from .models import TechModel, ProductModel, GadgetModel, FeedbackModel
# from .forms import Feedbackform  # Make sure you have imported your form

# def Feedback(request, pk):
#     user_id = request.session.get('user_id')
#     tech_id = request.session.get('tech_id')
    
#     instance1 = get_object_or_404(TechModel, pk=tech_id)
#     product = get_object_or_404(ProductModel, id=pk)
    
#     instance = get_object_or_404(GadgetModel, pk=user_id)
    
#     if request.method == 'POST':
#         form = Feedbackform(request.POST)
#         rating = request.POST.get('rating')  # Ensure this key matches your form field
        
#         if form.is_valid() and rating:
#             feedback_instance = form.save(commit=False)
#             feedback_instance.Userid = instance
#             feedback_instance.productid = product
#             feedback_instance.Techid = instance1
#             feedback_instance.Rating = int(rating)  # Ensure rating is saved as an integer
#             feedback_instance.save()
#             return redirect('prouserview')  # Redirect to the desired view after saving
#     else:
#         form = Feedbackform()
    
    # Fetch all ratings for the product
    # ratings = FeedbackModel.objects.filter(productid=product)
    # if ratings.exists():
    #     average_rating = ratings.aggregate(Avg('Rating'))['Rating__avg']
    # else:
    #     average_rating = None
    
    # return render(request, 'feedback.html', {
    #     'form': form,
    #     'product': product,
    #     'ratings': ratings,
    #     'average_rating': average_rating,
    # })

def Admin_login(request):
    
        if request.method=='POST':
            form=Adminloginform(request.POST)
            if form.is_valid():
                email=form.cleaned_data['Email']
                password=form.cleaned_data['Password']
                try:
                    user=AdminModel.objects.get(Email=email,Password=password)
                    return redirect('adminpage')
                except form.DoesNotExist:
                    form.add_error(None,"invalid email or password")
                    
        else:
            form=Adminloginform()
        return render(request,'adminlogin.html',{'form':form}) 
from django.http import JsonResponse
from django.db.models import Avg    
# def Feedback(request,pk,Techid):
#     var1=request.session.get('user_id')
   
#     instance1=get_object_or_404(GadgetModel,id=var1)
#     product = get_object_or_404(ProductModel,id=pk)
#     instance=get_object_or_404(TechModel,id=Techid)
#     ratings = FeedbackModel.objects.all()
#     average_rating = ratings.aggregate(FeedbackModel.Avg('rating'))['rating__avg']

#     if request.method == 'POST':
#         form = Feedbackform(request.POST)
#         if form.is_valid():
#             var1=form.save(commit=False)
         
#             var1.Userid=instance1
#             var1.productid=product
#             var1.Techid=instance
#             var1.save()
#             if request.is_ajax():
#                 return JsonResponse({'success': True})
#             else:
#                 return redirect('sucess') 
#     else:
#         if request.is_ajax():
#                 return JsonResponse({'success': False, 'errors': form.errors})
#         form = Feedbackform()

#     return render(request, 'feedback.html', {'form': form,'ratings': ratings,
#     'average_rating': average_rating})
def Feedback(request, pk, Techid):
    var1 = request.session.get('user_id')
   
    instance1 = get_object_or_404(GadgetModel, id=var1)
    product = get_object_or_404(ProductModel, id=pk)
    instance = get_object_or_404(TechModel, id=Techid)
    

    if request.method == 'POST':
        form =Feedbackform(request.POST)
        if form.is_valid():
            var1 = form.save(commit=False)
           
            var1.Userid = instance1
            var1.productid = product
            var1.Techid = instance
            var1.save()
            
           
            return redirect('fsuccess_page')
        
    else:
        form = Feedbackform()

    return render(request, 'feedback.html', {'form': form,})


def CCompliant(request, pk, Techid):
    var1 = request.session.get('user_id')
    instance1 = get_object_or_404(GadgetModel, id=var1)
    product_instance = get_object_or_404(ProductModel, id=pk)  # Correctly assign this instance
    technician_instance = get_object_or_404(TechModel, id=Techid)

    if request.method == 'POST':
        form = Compliants(request.POST)
        if form.is_valid():
            complaint = form.save(commit=False)
            complaint.Userid = instance1
            complaint.productid = product_instance  # Assign ProductModel instance
            complaint.Techid = technician_instance
            complaint.save()
            return redirect('csuccess_page')
    else:
        form = Compliants()

    return render(request, 'compliant.html', {'form': form, 'product': product_instance})



def psuccess_page(request):
    return render(request, 'paymentsucces.html')
def productsuccess_page(request):
    return render(request, 'productsucces.html')
def fsuccess_page(request):
    return render(request, 'feedbacksucces.html')
def csuccess_page(request):
    return render(request, 'compliantsucces.html')
def success_page(request):
    return render(request, 'regsuccess.html')
def usuccess_page(request):
    return render(request, 'userprofsucess.html')
def tsuccess_page(request):
    return render(request, 'techprofsuces.html')


import json

from django.http import JsonResponse
import json

def rate_tech(request, pk):
    if request.method == 'POST':
        data = json.loads(request.body)
        rating = data.get('rating')
        
        if rating is not None:
            try:
                product = ProductModel.objects.get(pk=pk)
                product.rating = int(rating)
                product.save()
                return JsonResponse({'status': 'success'}, status=200)
            except ProductModel.DoesNotExist:
                return JsonResponse({'status': 'error', 'message': 'Product not found'}, status=404)
        else:
            return JsonResponse({'status': 'error', 'message': 'Invalid rating'}, status=400)
    return JsonResponse({'status': 'error', 'message': 'Invalid request method'}, status=405)



import random
from django.core.mail import send_mail
from .forms import PasswordResetRequestForm
from .forms import OTPVerificationForm
from .forms import SetNewPasswordForm
def send_otp(Email):
    otp = random.randint(100000, 999999)  # Generate a 6-digit OTP
    if GadgetModel.objects.filter(Email=Email):
        gadgetotp=GadgetModel.objects.filter(Email=Email).first()
        gadgetotp.otp=otp
        gadgetotp.save()
    else:
        gadgetotp=TechModel.objects.filter(Email=Email).first()
        gadgetotp.otp=otp
        gadgetotp.save()
        
    subject = 'Your Password Reset OTP'
    message = f'Your OTP for password reset is {otp}.'
    send_mail(subject, message, 'ammusashok2018@gmail.com',[Email])
    return otp
def get_user_by_email(Email):
    try:
        return GadgetModel.objects.get(Email=Email)
    except GadgetModel.DoesNotExist:
        pass

    try:
        return TechModel.objects.get(Email=Email)
    except TechModel.DoesNotExist:
        pass
    return None
def password_reset_request(request):
    if request.method == 'POST':
        form = PasswordResetRequestForm(request.POST)
        if form.is_valid():
            Email = form.cleaned_data['Email']
            user = get_user_by_email(Email)
            if user:
                otp = send_otp(Email)
                request.session['otp'] = otp
                request.session['Email'] = Email
                return redirect('password-resetverify')
            else:
                form.add_error('email', 'Email does not exist.')
    else:
        form = PasswordResetRequestForm()
    return render(request, 'password_reset_request.html', {'form': form})

def password_reset_verify_otp(request):
    if request.method == 'POST':
        form = OTPVerificationForm(request.POST)
        if form.is_valid():
            entered_otp = form.cleaned_data['otp']
            saved_otp = request.session.get('otp')
            if entered_otp == str(saved_otp):
                return redirect('password-resetnew')
            else:
                form.add_error('otp', 'Invalid OTP.')
    else:
        form = OTPVerificationForm()
    return render(request, 'password_reset_verify_otp.html', {'form': form})

def password_reset_form(request):
    if request.method == 'POST':
        form = SetNewPasswordForm(request.POST)
        if form.is_valid():
            new_password = form.cleaned_data['new_password']
            Email = request.session.get('Email')
            user = get_user_by_email(Email)
            if user:
                user.Password = new_password
                user.save()
                user_type = user.usertype  # Retrieve the usertype from the user object
                return redirect(f'/loginuser/{user_type}')
            else:
                form.add_error(None, 'Something went wrong.')
    else:
        form = SetNewPasswordForm()
    return render(request, 'password_reset_form.html', {'form': form})

# from .forms import PasswordResetRequestForm
# from .forms import OTPVerificationForm
# from .forms import SetNewPasswordForm
# def send_otp(Email):
#     otp = random.randint(100000, 999999)  # Generate a 6-digit OTP
#     subject = 'Your Password Reset OTP'
#     message = f'Your OTP for password reset is {otp}.'
#     send_mail(subject, message, 'your-email@example.com', [Email])
#     return otp
# def get_user_by_email(Email):
#     try:
#         return GadgetModel.objects.get(Email=Email)
#     except GadgetModel.DoesNotExist:
#         pass
#     except GadgetModel.MultipleObjectsReturned:
#         # Handle the scenario where multiple records are found
#         return GadgetModel.objects.filter(Email=Email).first()

#     try:
#         return TechModel.objects.get(Email=Email)
#     except TechModel.DoesNotExist:
#         pass
#     except TechModel.MultipleObjectsReturned:
#         # Handle the scenario where multiple records are found
#         return TechModel.objects.filter(Email=Email).first()
    
#     return None

#     return None


# def password_reset_request(request):
#     if request.method == 'POST':
#         form = PasswordResetRequestForm(request.POST)
#         if form.is_valid():
#             Email = form.cleaned_data['Email']
#             user = get_user_by_email(Email)
#             if user:
#                 otp = send_otp(Email)
#                 request.session['otp'] = otp
#                 request.session['Email'] = Email
#                 return redirect('password_reset_verify_otp')
#             else:
#                 form.add_error('email', 'Email does not exist.')
#     else:
#         form = PasswordResetRequestForm()
#     return render(request, 'password_reset_request.html', {'form': form})

# def password_reset_verify_otp(request):
#     if request.method == 'POST':
#         form = OTPVerificationForm(request.POST)
#         if form.is_valid():
#             entered_otp = form.cleaned_data['otp']
#             saved_otp = request.session.get('otp')
#             if entered_otp == str(saved_otp):
#                 return redirect('password_reset_form')
#             else:
#                 form.add_error('otp', 'Invalid OTP.')
#     else:
#         form = OTPVerificationForm()
#     return render(request, 'password_reset_verify_otp.html', {'form': form})

# def password_reset_form(request):
#     if request.method == 'POST':
#         form = SetNewPasswordForm(request.POST)
#         if form.is_valid():
#             new_password = form.cleaned_data['new_password']
#             Email = request.session.get('Email')
#             user = get_user_by_email(Email)
#             if user:
#                 user.password = new_password
#                 user.save()
#                 user_type = user.usertype  # Retrieve the usertype from the user object
#                 return redirect(f'/loginuser/{user_type}')
#             else:
#                 form.add_error(None, 'Something went wrong.')
#     else:
#         form = SetNewPasswordForm()
#     return render(request, 'password_reset_form.html', {'form': form})
from django.contrib.auth import logout
def logout_view(request):
    logout(request)  
    return redirect('index')

