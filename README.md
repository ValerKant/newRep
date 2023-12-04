# newRep
//


import SwiftUI

struct StickerEraserView: View {
    
    @EnvironmentObject private var store: AppStore
    @Environment(\.dismiss) private var dismiss
    
    private let sticker: Sticker
    
    @State private var maskEditor = MaskEditor(
        context: SharedContext.context,
        settings: .init(size: 15, softness: 5),
        maskImage: .create(from: .clear, size: .init(side: 512))
    )
    
    init(sticker: Sticker) {
        self.sticker = sticker
        
        _maskEditor = State(initialValue: MaskEditor(
            context: SharedContext.context,
            settings: .init(size: 15, softness: 5),
            maskImage: sticker.mask ?? .create(
                from: .clear,
                size: .init(side: 512)
            )
        ))
    }
    
    var body: some View {
        VStack {
            topBar
            GeometryReader { geo in
                Image(uiImage: sticker.image.masked(maskEditor.maskImage))
                    .resizable()
                    .scaledToFit()
                    .gesture(DragGesture(minimumDistance: 0)
                        .onChanged { point in
                            maskEditor.draw(to: CGPoint(
                                x: point.location.x / geo.size.width,
                                y: point.location.y / geo.size.height
                            ))
                        }
                        .onEnded{ point in
                            maskEditor.end(in: CGPoint(
                                x: point.location.x / geo.size.width,
                                y: point.location.y / geo.size.height
                            ))
                        }
                    )
                
            }
            .aspectRatio(1, contentMode: .fit)
            .padding()
            
            Spacer()
            sliders
            Spacer()
        }
    }
    
    private var sliders: some View {
        Group {
            CommonSliderView(value: $maskEditor.settings.size,
                             range: 0...50,
                             title: "Size")
            
            CommonSliderView(value: $maskEditor.settings.softness,
                             range: 0...20,
                             title: "Softness")
        }
        .padding(.horizontal)
    }
    
    private var topBar: some View {
        VStack {
            HStack {
                Button {
                    dismiss()
                } label: {
                    Text("Cancel")
                }
                Spacer()
                Button {
                    store.dispatch(.changeCollage(.changeSticker(
                        .changeMask(maskEditor.maskImage),
                        id: sticker.id
                    )))
                    dismiss()
                } label: {
                    Image(systemName: "")
                    Text("Apply")
                }
            }
            .padding(20)
        }
    }
    
}

struct StickerEraserView_Previews: PreviewProvider {
    static var previews: some View {
        StickerEraserView(sticker: StickersProvider.allStickers.first!)
            .environmentObject(AppStore.preview)
    }
}

