var assessmentAnswers;
var qs;
var queryString = new URLSearchParams(document.location.search);

function showWheel() {
    $(".questions, .stepIndicators").hide();
    $(".results").fadeIn();
}

$(function () {
    //check to see if coming from a share link. If so, immediately show wheel
    if (queryString.toString().length > 100) {
        updateWheel(queryString);
        $("#mainImage").hide();
        showWheel();
    }

    $(".nxtBtn").on("click", function (e) {
        e.preventDefault();
        $("#mainImage").hide();
        var nextFieldset = $(this).parents("fieldset").next("fieldset");
        $(this).parents("fieldset").fadeOut(function () {
            $(nextFieldset).slideDown();
            $(".crumb").removeClass("active");
            $(".crumb").eq($("#assessment fieldset:visible").index()).addClass("active");
            $(".assessmentBreadcrumb").siblings("h2").text($(".crumb.active").text());
        });
    });

    $(".subBtn").on("click", function (e) {
        e.preventDefault();
       
        // submit lead form data
        var leadFormData = new FormData();

        console.log(leadFormData);

        var leadFormIsValid = true;

        // check validity of form data and add to post
        for (var i = 0; i < $(".leadForm input").length; i++) {
            var formInput = $(".leadForm input").eq(i);
            if (!$(formInput)[0].checkValidity()) {
                leadFormIsValid = false;
            }
            leadFormData.append($(formInput).attr("id"), $(formInput).val());
        }

        if (leadFormIsValid) {
            console.log('lead form is valid');

            // form is valid so hide questions and show wheel results
            $(".questions").slideUp(function () {
                $(".stepIndicators").hide();
                $(".results").fadeIn();
            });

            console.log($("#assessment").serialize());

            assessmentAnswers = $("#assessment").serialize();
            qs = new URLSearchParams(assessmentAnswers);

            console.log(qs);

            updateWheel(qs);

            //add link back to finished brand wheel
            leadFormData.append("field5", "A new brand assessment submission");
            leadFormData.append("field6", document.location.href + "?" + qs.toString());
            $.ajax({
                type: "POST",//From the form method
                url: '/MailForm/Submit/534?pageID=42',//The URL of the form action  /Module/Render/378?pageID=138
                data: leadFormData, // serializes the form's elements.
                processData: false,
                contentType: false,
                success: function (data) {
                    // redirect to thank-you page
                    console.log("lead form successfully sent");
                }
            });
        }
        else {
            $(".leadForm input").each(function () {
                $(this)[0].reportValidity();
            });
        }

    });

    $(".prevBtn").on("click", function (e) {
        e.preventDefault();
        var prevFieldset = $(this).parents("fieldset").prev("fieldset");
        console.log("fieldset index = " + $(prevFieldset).index());
        $(this).parents("fieldset").fadeOut(function () {
            $(prevFieldset).slideDown();
            $(".crumb").removeClass("active");
            $(".crumb").eq($("#assessment fieldset:visible").index()).addClass("active");
        }
        );
    });

    $("#bWheel [id^='shape-']").on("click", function () {
        $(".well.description").fadeIn();
        var shape = $(this).attr("id").split("shape-").pop();
        $(".well").html("<h4 class='text-primary'>" + shape.replace('--', '/').replace(/-/g, ' ') + "</h4><p>" + $("input[name=" + shape + "]").parents(".form-group").children("label").text() + "</p>");
    });
});

function updateWheel (answers) {
    console.log('update wheel running...');
    console.log(answers);

    answers.forEach(function (value, key) {
        //console.log("#bWheel #shape-" + key + " gets this class " + value);
        $("#bWheel #shape-" + key + " path").attr("class", value)
        $("#bWheel #shape-" + key + " circle").attr("class", value)
    });

    $(".shareWheel").attr("href", "mailto:contact@backstory.com?subject=Check+out+my+Brand+Wheel&body=" + encodeURIComponent(document.location.href + "?" + answers.toString()));
}

