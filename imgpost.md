import React, { useRef, useState, ChangeEvent, useEffect } from 'react'; // React とフックをインポート
// 画像アセットのインポート
import daikon1 from '@/assets/images/daikon1.jpg';
import daikon2 from '@/assets/images/daikon2.jpg';
import daikon3 from '@/assets/images/daikon3.jpg';
import daikon4 from '@/assets/images/daikon4.jpg';
import daikon5 from '@/assets/images/daikon5.jpg';


// リストアイテムのデータの型定義
type Item = {
  id: number; // 各アイテムを一位に識別するための永続的なID
  title: string; // アイテムの初期タイトル (この実装では動的に上書きされるため、初期値は空でも良い)
  imgUrl: string; // アイテムの画像URL
};

// 挿入位置インジケーター（横線）の状態を表す型定義
type DropIndicator = {
  index: number; // インジケーターが表示される対象アイテムの現在のリスト内インデックス
  position: 'before' | 'after'; // 対象アイテムの前('before')か後('after')か
};

// メインのアプリケーションコンポーネント
const ImagePost = () => {
  // ドラッグ操作に関する状態と参照
  const dragItemOriginalIndexRef = useRef<number | null>(null); // ドラッグ開始時のアイテムの元の配列内インデックスを保持
  const [draggingItemId, setDraggingItemId] = useState<number | null>(null); // 現在ドラッグ中のアイテムの永続的なID (item.id)
  const [dropIndicator, setDropIndicator] = useState<DropIndicator | null>(null); // 挿入インジケーター（横線）の状態

  // ドラッグ＆ドロップで並び替えるアイテムのリストデータ
  const [data, setData] = useState<Item[]>([
    { id: 1, title: "", imgUrl: daikon1 }, // idはユニークな値を設定。titleは後で動的に設定するため初期値は空。
    { id: 2, title: "", imgUrl: daikon2 },
    { id: 3, title: "", imgUrl: daikon3 },
    { id: 4, title: "", imgUrl: daikon4 },
    { id: 5, title: "", imgUrl: daikon5 },
  ]);

  // 選択されたファイルオブジェクトを保持するstate
  // const [selectedFile, setSelectedFile] = useState<File | null>(null);
  // 生成された画像プレビュー用のURLを保持するstate
  // const [previewUrl, setPreviewUrl] = useState<string | null>(null);


  const handleFilesChange = (event: ChangeEvent<HTMLInputElement>) => {
    // ファイルが選択されているか確認
    if (event.target.files === null) {
      return;
    }

    if (data.length + event.target.files.length > 5) {
      alert('画像は最大5枚までしか選択できません。');
      event.target.value = ''; // inputの値をリセット（同じファイルを再度選択できるようにするため）
      return;
    }

    // 画像ファイルのみフィルタリング
    const imageFiles = Array.from(event.target.files).filter((file) => file.type.startsWith('image/'));
    if (imageFiles.length < event.target.files.length) {
      alert('選択できるのは画像ファイルのみです。');
      event.target.value = ''; // inputの値をリセット（同じファイルを再度選択できるようにするため）
      return;
    }

    const newItems = imageFiles.map((file) => {
      const objectUrl = URL.createObjectURL(file); // Fileオブジェクトから一時的なURLを生成
      const randomId = Math.floor(Math.random() * 100000);
      return { id: randomId, title: "", imgUrl: objectUrl }; // 新しいアイテムを作成
    });

    setData([...data, ...newItems]); // 新しいアイテムを既存のデータに追加
    event.target.value = ''; // inputの値をリセット（同じファイルを再度選択できるようにするため）
  };






  // ドラッグ開始時の処理
  const handleDragStart = (event: React.DragEvent<HTMLDivElement>, index: number, itemActualId: number) => {
    dragItemOriginalIndexRef.current = index; // ドラッグされたアイテムの現在のインデックスを保存
    setDraggingItemId(itemActualId); // ドラッグ中のアイテムの永続IDをstateに保存 (スタイル変更用)
    event.dataTransfer.effectAllowed = 'move'; // ドラッグ操作の種類を 'move' (移動) に設定
    // event.dataTransfer.setData('text/plain', ''); // Firefoxでのドラッグを有効にするために必要な場合がある
    // 必要であれば、ドラッグゴーストイメージをカスタマイズ
    event.dataTransfer.setDragImage(event.currentTarget, 50, 50); // ドラッグ中のイメージを設定 (50px, 50px の位置に設定)
  };

  // アイテムが他のアイテムの上にドラッグされている間の処理
  const handleDragOver = (event: React.DragEvent<HTMLDivElement>, targetItemIndex: number) => {
    event.preventDefault(); // デフォルトの動作（ドロップ拒否）をキャンセルし、ドロップを許可する

    const draggingItemOriginalIdx = dragItemOriginalIndexRef.current; // ドラッグ中のアイテムのインデックス

    // ドラッグ中でない、または自分自身の上にある場合は何もしない
    if (draggingItemOriginalIdx === null) {
      if (dropIndicator !== null) setDropIndicator(null); // インジケーターが表示されていれば消す
      return;
    }
    if (draggingItemOriginalIdx === targetItemIndex) {
      if (dropIndicator !== null) setDropIndicator(null); // 自分自身の上ではインジケーターを表示しない
      return;
    }

    // ドロップターゲット要素とマウスカーソルの位置を取得
    const targetElement = event.currentTarget as HTMLElement; // イベントが発生した要素 (ドロップターゲット候補)
    const rect = targetElement.getBoundingClientRect(); // 要素の位置とサイズを取得
    const clientY = event.clientY; // マウスカーソルのY座標

    // マウスカーソルがターゲット要素の上半分にあるか下半分にあるかを判定
    const isOverFirstHalf = clientY < rect.top + rect.height / 2;

    // 新しいインジケーターの状態を作成
    const newIndicator: DropIndicator = isOverFirstHalf
      ? { index: targetItemIndex, position: 'before' } // 上半分ならターゲットの前に挿入
      : { index: targetItemIndex, position: 'after' };  // 下半分ならターゲットの後に挿入

    // インジケーターの状態が実際に変更された場合のみ更新 (不要な再レンダリングとちらつきを防ぐ)
    if (dropIndicator?.index !== newIndicator.index || dropIndicator?.position !== newIndicator.position) {
      setDropIndicator(newIndicator);
    }
  };

  // アイテムがドロップされた時の処理
  const handleDrop = (event: React.DragEvent<HTMLDivElement>) => {
    // event.preventDefault(); // onDragEndでは通常不要だが、状況により必要になることも

    // ドラッグ中のアイテムや挿入位置情報がない場合は処理を中断
    if (dragItemOriginalIndexRef.current === null || dropIndicator === null) {
      resetDragState(); // ドラッグ関連の状態をリセット
      return;
    }

    const draggedItemOriginalIdx = dragItemOriginalIndexRef.current; // ドラッグされたアイテムの元のインデックス
    const { index: indicatorTargetItemIndex, position: indicatorPosition } = dropIndicator; // 挿入インジケーターの位置情報

    // 元のリストのコピーを作成して変更を行う (イミュータブルな更新)
    const newList = [...data];

    // 1. ドラッグされたアイテムをリストから一時的に取り出す
    const [draggedItem] = newList.splice(draggedItemOriginalIdx, 1);

    // 2. 挿入先のインデックスを決定
    let finalDropIndex = indicatorTargetItemIndex;
    if (indicatorPosition === 'after') {
      // 'after' の場合は、ターゲットアイテムの次の位置に挿入するのでインデックスを+1
      finalDropIndex = indicatorTargetItemIndex + 1;
    }

    // 3. ドラッグ元アイテムを削除したことによるインデックスのズレを補正
    //    もしドラッグ元のインデックスが、計算された挿入先のインデックスよりも前だった場合、
    //    spliceで要素が1つ取り除かれた分、挿入先のインデックスも1つ前にズレるので、それを補正する。
    if (draggedItemOriginalIdx < finalDropIndex) {
      finalDropIndex--;
    }

    // 4. 取り出したアイテムを新しい位置に挿入
    newList.splice(finalDropIndex, 0, draggedItem);

    // 5. 更新されたリストでstateを更新し、画面を再レンダリング
    setData(newList);

    // 6. ドラッグ関連の状態をリセット
    resetDragState();
  };

  // ドラッグ操作に関連する状態を初期状態にリセットするヘルパー関数
  const resetDragState = () => {
    dragItemOriginalIndexRef.current = null;
    setDraggingItemId(null);
    setDropIndicator(null);
  };

  const deleteImg = (event: React.MouseEvent<HTMLDivElement>, itemActualId: number) => {
    event.stopPropagation(); // イベントのバブリングを防ぐ

    // filter() を使って、指定したid以外の要素で新しい配列を作成
    const newArray = data.filter(item => item.id !== itemActualId);
    setData(newArray); // 新しい配列でstateを更新
  };

  // コンポーネントのレンダリング
  return (
    // アプリケーション全体のコンテナ。ドラッグリーブイベントを設定
    <div  className="border-b border-gray-500 col-span-9 p-4 flex flex-col items-start">
      {/* <div className='h-1'></div> */}



      {/* データリストをループして各アイテムをレンダリング */}
      {data.map((item, index) => {
        // 1. 表示するタイトルを決定: 最初のアイテム (index === 0) なら "1番！", それ以外は空
        const currentTitle = index === 0 ? "投稿サムネ" : "";
        // 2. 表示するIDを決定: 現在のインデックス + 1 (1始まりの連番)
        const displayId = index + 1;

        return (
          // React.Fragment を使用して、各アイテムとその前後の挿入線をグループ化
          // key には item.id (永続的でユニークなID) を使用し、Reactが効率的に差分更新できるようにする
          <React.Fragment key={item.id}>
            {/* 「前」の挿入線: dropIndicatorが現在のアイテムの前を示している場合に表示 */}
            {dropIndicator?.index === index && dropIndicator?.position === 'before' && (
              <div className="h-16" />
              // <div className="border-2 border-gray-300 h-40 bg-gray-200 my-1 w-1/4 transition-all duration-150" />
            )}



            <div
              className={`relative flex my-1 ml-1 w-52 h-32 ${draggingItemId === item.id ? 'opacity-50 cursor-grabbing' : 'cursor-grab' // ドラッグ中は半透明にし、カーソルを変更
                }`}
              onDragStart={(e) => handleDragStart(e, index, item.id)} // ドラッグ開始時のイベントを設定
              onDragOver={(e) => handleDragOver(e, index)}   // 要素が他の要素の上にドラッグされている間のイベントを設定
              onDragEnd={handleDrop}     // ドラッグ終了時のイベントを設定 (ドロップ時も含む)
              draggable={true}          // HTMLのdraggable属性をtrueに設定
            >



                {/* タイトル表示 */}
                {/* <p className="font-semibold">{currentTitle}</p> */}
                {/* 表示用IDの表示 */}
                <div className=" absolute text-2xl font-bold text-white  top-1 left-1 rounded-full  px-2">
                  {displayId}
                  {/* {displayId === 1 && (
                    <span className="text-sm ml-3 font-bold text-white">見出し</span>
                  )} */}
                </div>
                {/* <p className="text-xs text-gray-400">内部ID: {item.id}</p> デバッグ用に元のIDを表示したい場合 */}




              {item.imgUrl && (

                  <img
                    src={item.imgUrl}
                    alt="選択された画像のプレビュー"
                    className='w-full object-cover overflow-hidden rounded'
                    // style={{ maxWidth: '300px', maxHeight: '300px', border: '1px solid #ccc' }}
                  />

              )}



              {/* {selectedFile && (
                  <div style={{ marginTop: '10px' }}>
                      <p>ファイル名: {selectedFile.name}</p>
                      <p>ファイルタイプ: {selectedFile.type}</p>
                      <p>ファイルサイズ: {(selectedFile.size / 1024).toFixed(2)} KB</p>
                      <p>ファイルサイズ: {selectedFile.size} KB</p>
                  </div>
              )} */}
                {/* 画像削除ボタン */}
                <div className='absolute text-white rounded-full top-0 right-0 hover:cursor-pointer hover:bg-gray-400 w-fit p-1' onClick={(e) => deleteImg(e, item.id)}>
                  <svg
                    xmlns="http://www.w3.org/2000/svg"
                    fill="none"
                    viewBox="0 0 24 24"
                    stroke-width="3"
                    stroke="currentColor"
                    className="w-5 h-5"
                  >
                    <path stroke-linecap="round" stroke-linejoin="round" d="M6 18L18 6M6 6l12 12" />
                  </svg>
                </div>
            </div>





            {/* 「後」の挿入線: dropIndicatorが現在のアイテムの後を示している場合に表示 */}
            {dropIndicator?.index === index && dropIndicator?.position === 'after' && (
              <div className="h-16" />
              // <div className="border-2 border-gray-300 h-40 bg-gray-200 my-1 w-1/4 transition-all duration-150" />
            )}
          </React.Fragment>
        );
      })}

      <div className='
        h-40
        relative
        bg-gray-100
        border-2 border-dashed border-gray-300
        rounded-lg
        flex flex-col items-center justify-center
        text-center
        p-4
        hover:bg-gray-200 hover:border-gray-400
        transition-colors duration-150 ease-in-out
      '>
        <p className="text-gray-600">
          ここに画像をドラッグ・アンド・ドロップするか、<br />
          クリックしてファイルをアップロードしてください。
        </p>
        <input
          type="file"
          multiple
          accept="image/*" // imageファイルのみ選択できるようにフィルタリング
          onChange={handleFilesChange}
          className='absolute inset-0 w-full h-full opacity-0 cursor-pointer' // w-full h-full を明示的に追加
        />
      </div>

    </div>
  );
}

export default ImagePost;
